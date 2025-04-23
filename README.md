# fasttext-windows-wheels

**Pre-built wheel distributions of the [fasttext](https://github.com/facebookresearch/fastText) library for Windows!**  
✅ Supports `numpy > 2`  
⭐ **If this project was helpful to you, please consider giving it a star!**

---

## 🚀 Quick Install

You can install the pre-built Windows wheel directly:

```bash
pip install https://github.com/yourusername/fasttext-windows-wheels/releases/download/v0.9.3/fasttext-0.9.3-cp310-cp310-win_amd64.whl
```

---

## 🧩 About the Build

This build is based on the original fasttext `v0.9.3` and includes full support for `numpy > 2`.

### 🔧 How the build was made:

1. Downloaded the source code for `fasttext==0.9.3`.
2. Installed **Microsoft C++ Build Tools** (part of Visual Studio).
3. Set the following environment variable to ensure C++17 is used:
   ```powershell
   $env:CL = "/std:c++17"
   ```
4. Modified `fasttext_pybind.cc` to enable MSVC compatibility:
   At the top of the file, added:
   ```cpp
   #ifdef _MSC_VER
   #include <BaseTsd.h>
   typedef SSIZE_T ssize_t;
   #endif
   ```

---

## 🧠 Fixes for Numpy > 2 Compatibility

### Replaced `getInputMatrix` binding with:

```cpp
.def(
    "getInputMatrix",
    [](fasttext::FastText& m) {
        std::shared_ptr<const fasttext::DenseMatrix> mm = m.getInputMatrix();

        if (!mm || mm->size(0) == 0 || mm->size(1) == 0) {
            return py::array_t<fasttext::real>();
        }

        const fasttext::real* original_data_ptr = mm->data();
        size_t rows = mm->size(0);
        size_t cols = mm->size(1);
        size_t num_elements = rows * cols;
        size_t data_size_bytes = num_elements * sizeof(fasttext::real);

        fasttext::real* copied_data_ptr = new fasttext::real[num_elements];
        std::memcpy(copied_data_ptr, original_data_ptr, data_size_bytes);

        py::capsule free_when_done(copied_data_ptr, [](void *f) {
            delete[] reinterpret_cast<fasttext::real *>(f);
        });

        return py::array_t<fasttext::real>(
            {rows, cols},
            copied_data_ptr,
            free_when_done
        );
    }
)
```

### Replaced `getOutputMatrix` binding with:

```cpp
.def(
    "getOutputMatrix",
    [](fasttext::FastText& m) {
        std::shared_ptr<const fasttext::DenseMatrix> mm = m.getOutputMatrix();

        if (!mm || mm->size(0) == 0 || mm->size(1) == 0) {
            return py::array_t<fasttext::real>();
        }

        const fasttext::real* original_data_ptr = mm->data();
        size_t rows = mm->size(0);
        size_t cols = mm->size(1);
        size_t num_elements = rows * cols;
        size_t data_size_bytes = num_elements * sizeof(fasttext::real);

        fasttext::real* copied_data_ptr = new fasttext::real[num_elements];
        std::memcpy(copied_data_ptr, original_data_ptr, data_size_bytes);

        py::capsule free_when_done(copied_data_ptr, [](void *f) {
            delete[] reinterpret_cast<fasttext::real *>(f);
        });

        return py::array_t<fasttext::real>(
            {rows, cols},
            copied_data_ptr,
            free_when_done
        );
    }
)
```

### In `FastText.py`, replaced all `np.array(arr, copy=False)` with:

```python
np.asarray(arr)
```

---

## 🛠️ Build the Wheel Yourself

If you'd like to build the wheel manually:

```bash
pip wheel --no-cache-dir --use-pep517 .
```

This will produce a `.whl` file like:

```
fasttext-0.9.3-cp310-cp310-win_amd64.whl
```

Then install it with:

```bash
pip install fasttext-0.9.3-cp310-cp310-win_amd64.whl
```

---

## 🙌 Credits

Original library: [facebookresearch/fastText](https://github.com/facebookresearch/fastText)
