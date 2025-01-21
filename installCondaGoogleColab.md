## Install `CONDA` In Google Colab
To install Conda in Google Colab, a Python package, condacolab, is used, which provides an easy way to install Conda. We first install the condacolab package using the pip command. 
```
!pip install -q condacolab
import condacolab
condacolab.install()
```

### Verify the `CONDA` Installation
```
!conda --version
```
