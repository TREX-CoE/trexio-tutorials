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


TREXIO API also provides functions to convert a given determinant into the list of occupied orbitals. This can be done either per spin component (`trexio.to_orbital_list`) or for both spin components simultaneously (`trexio.to_orbital_list_up_dn`). For example, taking the first determinant from the `det_list`, the conversion is performed as follows:


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
     [28, 63, 61]
    Binary  representation:      
     ['0b11100', '0b111111', '0b111101']
    Orbital list representation: 
     [  3   4   5  65  66  67  68  69  70 129 131 132 133 134]


**Reminder:** we use the `int64_num` to slice the `det_num[0]` list of integers since we represent a determinant using `int64_num` integer bit fields per spin component.

**Note:** all arrays returned by the `trexio` Python API are, in fact, instances of the `numpy` array class called `ndarray`. Because of that, we use the procedure below to check that two orbital lists are identical. For the Python-ic lists, one can simply use the `assert list1 == list2` statement.


```python
assert (orb_list_up == orb_list_up_check).all()
assert (orb_list_dn == orb_list_dn_check).all()
```

We have sucessfully written the list of the CI determinants. The `demo_file` can be closed now.


```python
demo_file.close()
```

## Writing arrays of the CI coefficients in the TREXIO file

Let's create an array of the CI coefficients. Again, we use random values in this tutorial. For the sake of simplicity, we will not use chunking for writing/reading the CI coefficients here.


```python
coefficients = [(randint(1,100)-randint(1,100))/100 for _ in range(det_num)]
```


We can finally write the CI coefficients as follows:


```python
offset_file = 0
with trexio.File(FILENAME, mode='w', back_end=BACK_END) as demo_file:
    trexio.write_determinant_coefficient(demo_file, offset_file, det_num, coefficients)
    print(f'Succesfully written {det_num} coefficients')
```

    Succesfully written 1000000 coefficients


Let's remove the produced file to clean the disk space. However, feel free to comment the code below if you intend to analyse the output data manually (e.g. using the `h5dump` utility).


```python
remove_file(FILENAME)
```

## Conclusion

In this Tutorial, you have created a TREXIO file using HDF5 back end and have written a number of molecular orbitals, a list of the CI determinants and CI coefficients. You have also learned how to read this data back from the TREXIO file.
