# deepspeech.pytorch

# TODO
* Add validation via a second loader
* Multi-gpu support, how do we get around passing hidden,cell states for data parallel?
* Best path decoding then validation for WER/CER
* Support LibriSpeech via multi-processed scripts
* Sometimes gradients go boom, add inf check to prevent meter from measuring this (and going boom as well)
* Similar convergence as to torch7 version on an4

Implementation of [Baidu Warp-CTC](https://github.com/baidu-research/warp-ctc) using pytorch.
Creates a network based on the [DeepSpeech2](http://arxiv.org/pdf/1512.02595v1.pdf) architecture, trained with the CTC activation function.

# Installation

Several libraries are needed to be installed for training to work. I will assume that everything is being installed in
an Anaconda installation on Ubuntu.

The dataloader relies on Neon, as a result Neon has to be installed. Installation instructions can be seen [here](docs.continuum.io/anaconda/install) Instructions to install Neon for Anaconda/Ubuntu below:

```
sudo apt-get install libcurl4-openssl-dev clang libopencv-dev libsox-dev
sudo apt-get install ffmpeg
git clone https://github.com/NervanaSystems/neon.git
cd neon && make sysinstall
```

Install pytorch if you haven't already:
```
conda install pytorch -c https://conda.anaconda.org/t/6N-MsQ4WZ7jo/soumith
```

Install this fork for Warp-CTC bindings:
```
git clone https://github.com/SeanNaren/ctc.git
cd ctc
mkdir build
cd build
cmake ..
make
cd ../python
sudo /home/<your.name>/anaconda2/bin/python setup.py install # e.g sudo /home/sean.narenthiran/anaconda2/bin/python setup.py install
```

# Usage

## Dataset

Currently only supports an4. To download and setup the an4 dataset run below command in the root folder of the repo:

```
python get_an4.py
python create_dataset_manifest.py --root_path dataset/
```

This will generate csv manifests files used to load the data for training.

Optionally, a noise dataset can be used to inject noise artifically into the training data. Just fill a folder with noise wav files you want to inject (a source of noise files is the [musan dataset](http://www.openslr.org/17/)) and run the below command:
```
python create_noise_manifest.py --root_path noise/ # or whatever you've named your noise folder
```

## Training

You need to find the maximum duration of the training and testing samples. The command below will iterate through the current
folder and find the longest duration:

```
find . -type f -name "*.wav" | xargs soxi -D | sort | tail -n 1
```

Afterwards you can run the training script.

```
python main.py --max_duration 6.4 # This is the default max duration (for an4)
```
