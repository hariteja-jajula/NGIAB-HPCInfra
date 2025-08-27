
# Running NextGen (ngen) on UA-HPC

This guide documents all the steps and dependencies required to build and run the **NextGen Hydrologic Framework (ngen)** from source on the UA-HPC cluster, including installing Boost, building ngen, and running it with the AWI dataset.

---

## 📋 Requirements

On **UA-HPC**, you will need the following modules and libraries:

* **GCC 11.2.0** (for modern C++/Fortran support)
* **CMake ≥ 3.20** (UA-HPC provides 4.0.3)
* **Python ≥ 3.6.8** (tested with 3.11.7 on UA-HPC)
* **Boost 1.79.0** (must be built manually, system Boost 1.66 is too old)
* **UDUNITS2** (available as a system library on UA-HPC)
* **Git** (for cloning externals via submodules)

Optional (depending on configuration flags in CMake):

* MPI (OpenMPI or MPICH)
* NetCDF / HDF5 (not required unless you enable `NGEN_WITH_NETCDF=ON`)
* SQLite (optional)

---

## 🔧 Installation Steps

### 1. Load Required Modules

Purge environment and load required compilers/tools:

```bash
module purge
module load gcc/11.2.0
module load cmake/4.0.3
module load python/3.11.7
```

---

### 2. Get the Source

Clone the ngen repository:

```bash
git clone https://github.com/NOAA-OWP/ngen.git
cd ngen
```

---

### 3. Install Boost 1.79.0 (Local Build)

System Boost is too old (1.66). Build Boost 1.79.0 locally:

```bash
cd $HOME/ngen/ngen/build
wget https://boostorg.jfrog.io/artifactory/main/release/1.79.0/source/boost_1_79_0.tar.gz
tar -xvzf boost_1_79_0.tar.gz
```

This extracts Boost into:

```
$HOME/ngen/ngen/build/boost_1_79_0
```

You do not need to fully compile Boost — headers are sufficient for ngen.

---

### 4. Build ngen

Create a build directory and run CMake with Boost path:

```bash
cd $HOME/ngen/ngen
mkdir build_ngen && cd build_ngen

cmake .. \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_COMPILER=$(which gcc) \
  -DCMAKE_CXX_COMPILER=$(which g++) \
  -DBOOST_ROOT=$HOME/ngen/ngen/build/boost_1_79_0 \
  -DBoost_NO_SYSTEM_PATHS=ON
```

Then build:

```bash
make -j 8
```

If successful, the `ngen` and partitionGenerator executables will be created in:

```
$HOME/ngen/ngen/build_ngen/
```

---

### 5. Verify Linking

Check that ngen is linked against GCC 11.2.0 libraries:

```bash
ldd ./ngen 
```

You should see:


```
        linux-vdso.so.1 (0x0000155555551000)
        libudunits2.so.0 => /lib64/libudunits2.so.0 (0x0000155555109000)
        libdl.so.2 => /lib64/libdl.so.2 (0x0000155554f05000)
        libpthread.so.0 => /lib64/libpthread.so.0 (0x0000155554ce5000)
        libutil.so.1 => /lib64/libutil.so.1 (0x0000155554ae1000)
        libstdc++.so.6 => /cm/local/apps/gcc/11.2.0/lib64/libstdc++.so.6 (0x00001555546cb000)
        libm.so.6 => /lib64/libm.so.6 (0x0000155554349000)
        libgcc_s.so.1 => /cm/local/apps/gcc/11.2.0/lib64/libgcc_s.so.1 (0x0000155554130000)
        libc.so.6 => /lib64/libc.so.6 (0x0000155553d6b000)
        libexpat.so.1 => /lib64/libexpat.so.1 (0x0000155553b2f000)
        /lib64/ld-linux-x86-64.so.2 (0x0000155555326000)

```

---

## ▶ Running ngen with AWI Dataset

### 1. Dataset Structure

Your dataset folder should look like:

```
AWI_16_10154200_009/
├── config/
│   ├── realization.json
│   ├── gage-10154200_subset.gpkg
│   ├── troute.yaml
├── forcings/
├── metadata/
├── outputs/   (created by ngen)
```

### 2. Run ngen

From the build directory:

```bash
cd $HOME/ngen/ngen/build_ngen

# Run with dataset root
./ngen $HOME/ngen/ngen/ngen_data/AWI_16_10154200_009/

# Or explicitly with realization.json
./ngen $HOME/ngen/ngen/ngen_data/AWI_16_10154200_009/config/realization.json
```

### 3. Output Location

Results are written to:

```
$HOME/ngen/ngen/ngen_data/AWI_16_10154200_009/outputs/
```

---

## ⚠ Troubleshooting

* **GLIBCXX errors** → Make sure `gcc/11.2.0` is loaded before running.
* **Numpy import error (pybind11)** → `unset PYTHONPATH` before running.
* **Boost not found** → Ensure you passed `-DBOOST_ROOT` to CMake.
* **No outputs** → Check your `realization.json` for correct model setup.

---

## ✅ Summary

* Build Boost 1.79.0 locally
* Load GCC 11.2.0, CMake, Python 3.11.7
* Configure with CMake, pointing to Boost root
* Run ngen using your AWI dataset’s config/realization.json
* Outputs go to the dataset’s `outputs/` folder

---


