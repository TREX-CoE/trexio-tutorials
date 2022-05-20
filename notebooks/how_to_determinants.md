# How to write the CI determinants and coefficients

This tutorial covers an advanced use case of the TREXIO library based on the Python API. At this point, it is assumed that the `trexio` Python package has been successfully installed on the user machine or in the virtual environment. If this is not the case, feel free to follow the [installation guide](https://github.com/TREX-CoE/trexio/blob/master/python/README.md).

## Importing TREXIO

First of all, let's import the TREXIO package.


```python
try:
    import trexio
except ImportError:
    raise Exception("Unable to import trexio. Please check that trexio is properly installed.")
```

## Creating a new TREXIO file

TREXIO currently supports two back ends for file I/O:

1. `TREXIO_HDF5`, which relies on extensive use of the [HDF5 library](https://portal.hdfgroup.org/display/HDF5/HDF5) and the associated binary file format. This back end is optimized for high performance but it requires HDF5 to be installed on the user machine.

2. `TREXIO_TEXT`, which relies on basic I/O operations that are available in the standard C library. This back end is not optimized for performance but it is supposed to work "out-of-the-box" since there are no external dependencies.

Armed with these definitions, let's proceed with the tutorial. The first task is to create a TREXIO file called `determinants_demo.h5` using `TREXIO_HDF5` back end. But first we have to remove the file if it already exists in the current directory


```python
import os

def remove_file(filename: str) -> None:
    """Remove a file."""
    try:
        os.remove(FILENAME)
    except:
        print(f'{FILENAME} does not exist.')
```


```python
BACK_END = trexio.TREXIO_HDF5
FILENAME = 'determinants_demo.h5'
```


```python
remove_file(FILENAME)
```

    determinants_demo.h5 does not exist.


## Writing a list of determinants in the TREXIO file

Prior to any work with the TREXIO library, we highly recommend users to read about the [TREXIO internal configuration](https://trex-coe.github.io/trexio/trex.html), which explains the structure of the wavefunction file (format). The reason is that the TREXIO API has a naming convention, which is based on the groups and variables that are pre-defined by the developers in the aforementioned configuration.

First of all, let's create an instance of the `trexio.File` class and open it for writing.


```python
demo_file = trexio.File(FILENAME, mode='w', back_end=BACK_END)
```

Before we start the full-scale I/O of the determinants, we need to write a few data pieces in the file. One of them is the `mo_num`, which will determine the internal storage of the `determinant_list` (i.e. how many 64-bit integers is needed to represent a single determinant).


```python
mo_num = 150
```


```python
trexio.write_mo_num(demo_file, mo_num)
```

The number of 64-bit integers depends on the `mo_num` as follows:


```python
int64_num_check = int((mo_num-1)/64) + 1
```

TREXIO API now provides a `trexio.get_int64_num` function to compute this value for you based on the `mo_num` value present in the file. It expects only `trexio.File` instance as an input. Let's check it:


```python
int64_num = trexio.get_int64_num(demo_file) 
assert int64_num == int64_num_check
```

We can now create a list of the CI determinants. For simplicity, the integers representing the determinants will be generated randomly.


```python
det_num = 1000000
```


```python
from random import randint

det_list  = [
    [randint(1,100) for _ in range(int64_num*2)]
    for _ in range(det_num)
]
```

**Note:** one determinant can be represented with `int64_num` integers per spin component as discussed above. The size is then doubled to accommodate both up-spin and down-spin (alpha and beta) electrons. The `det_num` value determines the total number of the CI determinants.

The API for the determinants I/O is very similar to the one used for `sparse` quantities (e.g. `rdm_2e`). The key difference is that there is only one array of values to be passed as an argument (i.e. no need to provide the indices from the `sparse` data representation). The determinants I/O is performed in chunks (just like `sparse` data), which is why the API call expects also the file offset and the chunk size as input arguments.


```python
n_chunks    = 5
chunk_size  = int(det_num/n_chunks)
```

We can now write the determinants `n_chunks` times. **Note:** do not forget to increment the `offset_file` value, otherwise the values will be overwritten instead of being appended.


```python
offset_file = 0
for _ in range(n_chunks):
    trexio.write_determinant_list(demo_file, offset_file, chunk_size, det_list[offset_file:])
    print(f'Succesfully written {chunk_size} determinants to file position {offset_file}')
    offset_file += chunk_size
```

    Succesfully written 200000 determinants to file position 0
    Succesfully written 200000 determinants to file position 200000
    Succesfully written 200000 determinants to file position 400000
    Succesfully written 200000 determinants to file position 600000
    Succesfully written 200000 determinants to file position 800000


TREXIO API also provides functions to convert a given determinant into the list of occupied orbitals. This can be done either per spin component (`trexio.to_orbital_list`) or for both spin components simultaneously (`trexio.to_orbital_list_up_dn`) as follows:


```python
up_spin_det = det_list[0][:int64_num]
dn_spin_det = det_list[0][int64_num:]

orb_list_up, orb_list_dn = trexio.to_orbital_list_up_dn(int64_num, det_list[0])

orb_list_up_check = trexio.to_orbital_list(int64_num, up_spin_det)
orb_list_dn_check = trexio.to_orbital_list(int64_num, dn_spin_det)

print("Integer representation:      \n", up_spin_det)
print("Binary  representation:      \n", [bin(i) for i in up_spin_det])
print("Orbital list representation: \n", orb_list_up)
```

    Integer representation:      
     [79, 1, 13]
    Binary  representation:      
     ['0b1001111', '0b1', '0b1101']
    Orbital list representation: 
     [  1   2   3   4   7  65 129 131 132]


**Reminder:** all arrays returned by the `trexio` Python API are, in fact, instances of the `numpy` array class called `ndarray`. Because of that, we use the procedure below to check that two orbital lists are identical. For the Python-ic lists, one can simply use the `assert list1 == list2` statement.


```python
assert (orb_list_up == orb_list_up_check).all()
assert (orb_list_dn == orb_list_dn_check).all()
```

We have sucessfully written the list of the CI determinants. The `demo_file` can be closed now.


```python
demo_file.close()
```

## Writing arrays of the CI coefficients in the TREXIO file

The CI calculation can be performed for ground and excited states. Thus, one might want to write/read these coefficients per state. The `trexio` API provides this functionality using the global state switch, which is controlled using the `trexio.set_state` function.

Let's create an array of the CI coefficients. Again, we use arbitrary values in this tutorial. For the sake of simplicity, we will not use chinking for writing/reading the CI coefficients here.


```python
coefficients = [3.14 + float(i) for i in range(det_num)]
```

Since we would like to perform an I/O for several states, we can prepare a bigger array containing CI coefficients for all states:


```python
n_states = 4
coefficients_all = [
    [coefficients[i]*float(j+1) for i in range(len(coefficients))]
    for j in range(n_states)
]
```

We can finally write the CI coefficients as follows:


```python
offset_file = 0
for s in range(n_states):
    with trexio.File(FILENAME, mode='w', back_end=BACK_END) as demo_file:
        demo_file.set_state(s)
        trexio.write_determinant_coefficient(
            demo_file, offset_file, det_num, coefficients_all[s]
        )
        print(f'Succesfully written {det_num} coefficients for state {s}')
```

    Succesfully written 1000000 coefficients for state 0
    Succesfully written 1000000 coefficients for state 1
    Succesfully written 1000000 coefficients for state 2
    Succesfully written 1000000 coefficients for state 3


## Reading the CI coefficients in serial and in parallel

Now, let's prepare a separate function to read the determinant coefficients. You will see later how this function can facilitate the parallel I/O using the built-in Python functionalities. 


```python
def read_coefficients (state: int, offset_file: int, det_num: int) -> list:
    with trexio.File(FILENAME, mode='r', back_end=BACK_END) as demo_file:
        demo_file.set_state(state)
        coefficients = trexio.read_determinant_coefficient(
            demo_file, offset_file, det_num
        )
        #print(f'Succesfully read {det_num} coefficients for state {state}\n')
        return coefficients
```

We can now read the data in parallel using the built-in Python modules like `multiprocessing`. Firstly, let's perform the serial reading:


```python
# Serial run

offset_file = 0
coefficients_read_all = []

for i in range(n_states):
    coefficients_read = read_coefficients(i, offset_file, det_num)
    coefficients_read_all.append(coefficients)

print(f'Serial read, {n_states} states: done')
```

    Serial read, 4 states: done



```python
# Parallel (MPI-like) run
from multiprocessing import Process

jobs        = []
offset_file = 0

for s in range(n_states):
    p = Process(
        target=read_coefficients,
        args=(s, offset_file, det_num)
    )
    jobs.append(p)
    p.start()

for j in jobs:
    j.join()

print(f'Parallel read, {n_states} states: done')
```

    Parallel read, 4 states: done


Let's remove the produced file to clean the disk space. However, feel free to comment the line below if you intend to analyse the output data manually (e.g. using the `h5dump` utility).


```python
remove_file(FILENAME)
```

## Conclusion

In this Tutorial, you have created a TREXIO file using HDF5 back end and have written a number of molecular orbitals, a list of the CI determinants and CI coefficients for several states. You have also learned how to read this data back from the TREXIO file in serial and in parallel.
