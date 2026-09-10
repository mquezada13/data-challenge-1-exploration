## Update: Fix for `ModuleNotFoundError: No module named 'ROOT'`

The notebook failed with:

```text
ModuleNotFoundError: No module named 'ROOT'
```

The problem was related to the ROOT installation used by the COSI environment.

Two ROOT installations were found on the system:

```text
Homebrew ROOT:
/opt/homebrew/Cellar/root/6.40.04

COSItools ROOT:
/Users/maura/Documents/COSI/COSItools/external/root_v6.40.4
```

The Homebrew ROOT installation was built against Python 3.14.7, while the original COSI Python environment uses Python 3.12.14. Therefore, the Homebrew ROOT installation is not compatible with the COSI environment.

The COSItools ROOT installation is the correct one for this project:

```text
/Users/maura/Documents/COSI/COSItools/external/root_v6.40.4
```

It was built against Python 3.12.14, which matches the existing COSI Python environment:

```text
/Users/maura/Documents/COSI/COSItools/python-env
```

The working dependency chain is therefore:

```text
COSItools/python-env
    ↓
Python 3.12.14
    ↓
COSItools/external/root_v6.40.4
    ↓
MEGAlib
    ↓
COSI Data Challenge 1
```

MEGAlib is located at:

```text
/Users/maura/Documents/COSI/COSItools/megalib
```

and its library is:

```text
/Users/maura/Documents/COSI/COSItools/megalib/lib/libMEGAlib.so
```

To start the COSI Data Challenge correctly, open a terminal and run the following commands:

```bash
cd /Users/maura/Documents/COSI/data-challenge-1

source /Users/maura/Documents/COSI/COSItools/python-env/bin/activate

source /Users/maura/Documents/COSI/COSItools/external/root_v6.40.4/bin/thisroot.sh

export MEGAlib=/Users/maura/Documents/COSI/COSItools/megalib
```

Check that the correct Python version is active:

```bash
python --version
```

Expected output:

```text
Python 3.12.14
```

Check that the correct ROOT installation is being used:

```bash
which root
```

Expected path:

```text
/Users/maura/Documents/COSI/COSItools/external/root_v6.40.4/bin/root
```

Check the MEGAlib environment variable:

```bash
echo $MEGAlib
```

Expected output:

```text
/Users/maura/Documents/COSI/COSItools/megalib
```

Check that ROOT can be imported from Python:

```bash
python -c "import ROOT; print(ROOT.__version__)"
```

Expected output:

```text
6.40.04
```

MEGAlib can also be tested directly with:

```bash
python - <<'PY'
import ROOT
print("ROOT:", ROOT.__version__)
print("Loading MEGAlib:", ROOT.gSystem.Load("$(MEGAlib)/lib/libMEGAlib.so"))
print("MGlobal:", hasattr(ROOT, "MGlobal"))
PY
```

The successful test produced:

```text
ROOT: 6.40.04
Loading MEGAlib: 0
MGlobal: True
```

A return value of `0` from `ROOT.gSystem.Load(...)` means that the MEGAlib library was loaded successfully.

`MGlobal: True` confirms that ROOT can access the MEGAlib `MGlobal` class required by `COSIpy_dc1.py`.

A CFITSIO warning may also appear:

```text
WARNING: version mismatch between CFITSIO header (...) and linked library (...).
```

This did not prevent ROOT or MEGAlib from loading, so it can be ignored unless a later FITS-related error appears.

### Important VS Code / Jupyter step

Even if ROOT works in the terminal, a VS Code Jupyter notebook may still report:

```text
ModuleNotFoundError: No module named 'ROOT'
```

This happens because VS Code/Jupyter may not inherit the ROOT and MEGAlib environment variables from an already running VS Code session.

Therefore, VS Code should be launched **from the same terminal where the COSI Python environment, ROOT, and MEGAlib have already been configured**.

The complete startup sequence is:

```bash
cd /Users/maura/Documents/COSI/data-challenge-1

source /Users/maura/Documents/COSI/COSItools/python-env/bin/activate

source /Users/maura/Documents/COSI/COSItools/external/root_v6.40.4/bin/thisroot.sh

export MEGAlib=/Users/maura/Documents/COSI/COSItools/megalib

python -c "import ROOT; print(ROOT.__version__)"

code .
```

The order is important:

```text
1. Go to the Data Challenge repository
2. Activate the COSI Python environment
3. Source the COSItools ROOT installation
4. Define the MEGAlib path
5. Verify that ROOT imports correctly
6. Launch VS Code from that same terminal with `code .`
```

Inside VS Code, select the following Python interpreter/kernel for the notebook:

```text
/Users/maura/Documents/COSI/COSItools/python-env/bin/python
```

If the notebook was already open, restart the notebook kernel after selecting the correct interpreter.

The interpreter used by the notebook can be checked with:

```python
import sys
print(sys.executable)
```

It should return:

```text
/Users/maura/Documents/COSI/COSItools/python-env/bin/python
```

### Important

Do **not** use the Homebrew ROOT installation for this project:

```text
/opt/homebrew/Cellar/root/6.40.04
```

That ROOT installation was built for Python 3.14.7 and is incompatible with the COSI Python 3.12.14 environment.

Using Homebrew ROOT together with the COSItools MEGAlib installation also caused duplicate ROOT libraries to be loaded simultaneously, producing errors such as:

```text
Class RunStopper is implemented in both ...
```

followed by:

```text
bus error
segmentation violation
```

This happened because MEGAlib was linked against the ROOT installation inside `COSItools`, while Python had loaded the Homebrew ROOT installation.

Therefore, the correct configuration for COSI Data Challenge 1 is:

```text
Python:
/Users/maura/Documents/COSI/COSItools/python-env/bin/python

ROOT:
/Users/maura/Documents/COSI/COSItools/external/root_v6.40.4

MEGAlib:
/Users/maura/Documents/COSI/COSItools/megalib

Repository:
/Users/maura/Documents/COSI/data-challenge-1
```

For future sessions, the recommended way to start working on the Data Challenge is simply:

```bash
cd /Users/maura/Documents/COSI/data-challenge-1

source /Users/maura/Documents/COSI/COSItools/python-env/bin/activate

source /Users/maura/Documents/COSI/COSItools/external/root_v6.40.4/bin/thisroot.sh

export MEGAlib=/Users/maura/Documents/COSI/COSItools/megalib

code .
```


## Git LFS: downloading large files in the repository

Some files in the COSI Data Challenge repository are stored using **Git LFS (Large File Storage)**.

Git LFS is used for large binary files such as:

- simulation files
- detector responses
- event data
- compressed datasets
- other large analysis products

Instead of storing the full file directly in normal Git history, Git LFS stores a small **pointer file** in the repository.

A Git LFS pointer looks like this:

```text
version https://git-lfs.github.com/spec/v1
oid sha256:...
size 73800035
```

If a supposed data file opens as plain text containing lines like these, then the real file has **not been downloaded yet**.

### Check that Git LFS is installed

From the repository terminal:

```bash
git lfs version
```

If Git LFS is installed correctly, this should print a version number.

### Download the large files

Go to the repository root:

```bash
cd /path/to/repository
```

Then download the Git LFS files associated with the current branch:

```bash
git lfs pull
```

This replaces the small pointer files with the actual large files.

### Check which files are managed by Git LFS

```bash
git lfs ls-files
```

### Check Git LFS status

```bash
git lfs status
```

### Check whether a file is still only a pointer

Inspect the beginning of the file:

```bash
head path/to/large_file
```

If the output starts with:

```text
version https://git-lfs.github.com/spec/v1
```

then the file is still only an LFS pointer and the real data have not been downloaded.

### Check the file size

```bash
ls -lh path/to/large_file
```

Before downloading, the pointer file may only be a few hundred bytes.

After:

```bash
git lfs pull
```

the file size should correspond to the real dataset, often tens or hundreds of MB.

### Typical workflow after cloning a repository

```bash
git clone <repository-url>

cd <repository-directory>

git lfs pull
```

### Typical workflow when the repository is already cloned

If the repository already exists locally but some large files are still pointers:

```bash
cd /path/to/repository

git lfs pull
```

### Other useful Git LFS command

Fetch the LFS objects without necessarily updating the working files:

```bash
git lfs fetch
```

### Important

A file can exist with the correct filename and still **not contain the real data**.

For example, a file such as:

```text
data/flight/Crab_COSIBalloonData.tra.gz
```

may appear in the repository but still contain only an LFS pointer.

Therefore, if an analysis:

- loads zero events
- produces an empty dataset
- fails while reading a large file
- gives unexpected file-format errors

check first whether the file is the real downloaded LFS object or only a Git LFS pointer.

The quickest diagnostic is:

```bash
head path/to/file
```

If it begins with:

```text
version https://git-lfs.github.com/spec/v1
```

run:

```bash
git lfs pull
```

from the repository root before debugging the analysis code itself.