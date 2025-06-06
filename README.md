<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->
[![Contributors][contributors-shield]][contributors-url]
[![Forks][forks-shield]][forks-url]
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![GNU GPL-v3.0][license-shield]][license-url]


<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://synthrad2025.grand-challenge.org/">
    <img src="./SynthRAD_banner.png" alt="Logo" width="770" height="160">
  </a>


  <p align="center">
    Algorithm template docker for submissions to
<a href="https://synthrad2025.grand-challenge.org/"><strong>SynthRAD2025 Grand Challenge</strong></a>
  <br />
    <a href="https://github.com/SynthRAD2025/algorithm-template"><strong>Explore the docs »</strong></a>
    <br />
    <br />
    <a href="https://github.com/SynthRAD2025/algorithm-template">View Demo</a>
    ·
    <a href="https://github.com/SynthRAD2025/algorithm-template/issues">Report Bug</a>
    ·
    <a href="https://github.com/SynthRAD2025/algorithm-template/issues">Request Feature</a>
  </p>
</p>

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [Goal](#goal)
* [Getting Started](#getting-started)
  * [Dependencies](#dependencies)
  * [Download](#download)

* [Usage](#usage)
  * [Using the template](#using-the-template)
  * [Making submissions](#making-submissions-to-task-1-and-2)
  * [Build and export docker](#building-and-exporting-the-docker-container-with-your-new-algorithm)
  * [Uploading on GC portal](#uploading-on-gc-portal)
  * [Adding model files](#adding-your-own-model-files-to-the-docker-container)
  * [Run docker locally with GPU support ](#to-run-your-docker-locally-with-gpu-support)

  * [Testing locally w/o docker](#testing-your-algorithm-locally-without-docker)
  * [Reformatting your data](#reformatting-your-own-data)






* [Roadmap](#roadmap)
* [Contributing](#contributing)
* [License](#license)
* [Contact](#contact)
<!--
* [Acknowledgements](#acknowledgements)
-->


<!-- ABOUT THE PROJECT -->
## Goal
The goal of this repository is to provide a seamless and efficient integration of your algorithm into the Grand Challenge platform. This repository offers a template for packaging your algorithm in a Docker container, ensuring compatibility with the input and output formats of the SynthRAD2025 challenge. 

With this template, you can submit your algorithm (with minimal effort) to the test phases of the challenge (and optionally the validation phases). You can use the preliminary test phase to make sure everything is in order with your submission and get an initial estimate of your algorithms performance. 

<!-- GETTING STARTED -->
## Getting Started

### Dependencies

For building the algorithm for submission, the user should have access to Docker [https://docs.docker.com/] on their system. (Note: Submissions to the test phase are allowed only with docker containers). This algorithm template was tested using Ubuntu 24.04 and Docker 28.1.1.

Please make sure to list the requirements for your algorithm in the `requirements.txt` file as this will be picked up when building the docker container. 


### Download

1. Clone the repo
```sh
git clone https://github.com/SynthRAD2025/algorithm-template.git
```
or
```sh
git clone git@github.com:SynthRAD2025/algorithm-template.git
```


### Setup

Dummy data (randomly created scans) for testing the docker has already been provided in the `test` folder. It should be in the following format,

```
algorithm-template
└── test
    ├── images
    │   ├── body
    │   │   └── 1HNAxxx.nii.gz
    │   └── mri
    │       └── 1HNAxxx.nii.gz
    ├── region.json
    └── expected_output.json
```

`test/images` simulates the input data provided to the docker image while it is run on the test data by the Grand Challenge platform. `test/expected_output.json` is present to check if your algorithm provides the expected output on the inputs provided to it. 


<!-- USAGE EXAMPLES -->

## Usage

First, run `test.sh`

`test.sh` will build the docker container (which contains the algorithm), provide the `test` folder as input to the docker container and run the algorithm. The output of the algorithm will be placed in the `output` directory. Tests will also be run to check if the algorithm provides the expected output.

Note: It is recommended to run this before you integrate your own algorithm into the template.

The output of the test.sh script should look similar to this:

```
########## ENVIRONMENT VARIABLES ##########
TASK_TYPE: mri
INPUT_FOLDER: /input
Scan region:  Head and Neck
[
    {
        "outputs": [
            {
                "type": "metaio_image",
                "filename": "/output/images/synthetic-ct/1BAxxx.nii.gz"
            }
        ],
        "inputs": [
            {
                "type": "metaio_image",
                "filename": "/input/images/mri/1BAxxx.nii.gz"
            },
            {
                "type": "metaio_image",
                "filename": "/input/images/body/1BAxxx.nii.gz"
            },
            {
                "type": "String",
                "filename": "/input/region.json"
            }
        ],
        "error_messages": []
    }
]
Tests successfully passed...
```

### Using the template
To integrate your algorithm into this template, you need to modify the `predict` function of the `SynthradAlgorithm` in the `process.py` file.
```{python}
class SynthradAlgorithm(BaseSynthradAlgorithm):
    ...

    def predict(self, input_dict: Dict[str, sitk.Image]) -> sitk.Image:
        """
        Your algorithm implementation.
        """
        # Your code here
        return output_image
```

You might need to load your models first before running the algorithm and this can be done in the `__init__` function of the `SynthradAlgorithm` class. For instance, to load a pytorch model,
```{python}
class SynthradAlgorithm(BaseSynthradAlgorithm):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

        # Load the PyTorch model
        model_path = os.path.join(os.path.dirname(__file__), 'your_model.pt')
        self.model = torch.load(model_path)
        self.model.eval()
```        


### NOTE
GrandChallenge processes algorithm dockers on a per-scan basis as mentioned [here](https://grand-challenge.org/documentation/choose-input-and-output-interfaces/). The `test` folder in this repo represents one such case and simulates how the algorithm docker will be run on a single case when evaluating on gc. If you would like to test on multiple cases, please create copies of the test repository and add each case's data into them. You could make a trial submission and look at [debug instructions](https://grand-challenge.org/forums/forum/synthrad2023-643/topic/debugging-algorithm-docker-submissions-after-submitting-on-the-challenge-1478/) to see exactly how it is run on a case-by-case basis. 

### NEW: Obtaining region information for the scan.
This is a newly added functionality based on requests from our participants. You can access a `"region"` key in the `input_dict` argument to the predict function. `process.py` shows a sample of how it can be obtained and used. 

### Making submissions to Task 1 and 2.
Since the challenge contains two tasks, you will need to provide separate docker containers for each (even if you run the exact same algorithm on both). To configure which task your docker will be built for, we have provided a `.env` file. You can modify it before building the docker image and your docker will be built for the selected task.
```
TASK_TYPE="cbct" # Set to mri (Task 1) or cbct (Task 2)
INPUT_FOLDER="/input" # Do not change unless you want to test locally
```

Change the `TASK_TYPE` to "cbct" or "mri" depending on the task you want to submit for. Do not change the `INPUT_FOLDER` unless you are testing locally.


### Building and exporting the docker container with your new algorithm!
It is recommended to run `test.sh` first as it will ensure that your new algorithm always runs as expected. 

1. Run the `export.sh` command. You can provide a name as the next argument for naming your docker containers. 
  Example: `./export.sh cbct_docker`

2. You might need to wait a bit for this process to complete (depending on your model size and dependencies added by you) as it builds and saves the docker as `.tar.gz`. Once you have this `.tar.gz` file, you can submit it on the grand challenge portal in the SynthRAD submissions! 

### Uploading on GC portal
Please find detailed video instructions @ https://www.youtube.com/watch?v=RYj9BOJJNV0

### Adding your own model files to the docker container
This step requires a bit of familiarity with the docker ecosystem as you will need to edit the `Dockerfile` to do so. The models will be embedded into the docker container allowing the docker to run independently on any system!

As a start, you can copy model files into the docker by adding something like this into the `Dockerfile`,
```
COPY --chown=algorithm:algorithm your_model.pt /opt/algorithm/

```

Once you do this, `your_model.pt` should be accessible in the `___init___` function as described above. 

### To run your docker locally with GPU Support
Ensure that you have the Nvidia-container-toolkit installed along with your docker installation. 

You can follow the instructions here: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html

Once you have this installed, you can run `./test_gpu.sh` instead of `test.sh` 

### Testing your algorithm locally (without docker).
To first test your algorithm locally (without the docker build process etc. - which significantly speeds up over multiple iterations),
1. Configure `.env` for local mode by setting the `INPUT_FOLDER` to the path where you want to provide the inputs from. For instance, the `test` folder is a good starting place. But you could also provide your own data. (!!! NOTE: SET THE `INPUT_FOLDER` back to `/input` before you build the docker)

2. Run `python process.py` in an environment with all your dependencies installed. 


This should run your algorithm locally and allow you to test different iterations before making a docker container.


### Reformatting your own data
In the `setup` folder, the `create_dummy_data.py` gives an example of how the dummy data is created for the docker container. You can reformat your data to be run by the docker container.

For the MRI task, this is how the data should be organized.
```
data
  ├── images
  │   ├── body
  │   │   └── 1HNAxxx.nii.gz
  │   │   └── ... 
  │   ├── mri
  │   │   └── 1HNAxxx.nii.gz
  │   │   └── ... 

```

Similarly, for the CBCT task,
```
data
  ├── images
  │   ├── body
  │   │   └── 1HNAxxx.nii.gz
  │   │   └── ... 
  │   ├── cbct
  │   │   └── 1HNAxxx.nii.gz
  │   │   └── ... 

```

<!-- ROADMAP -->
## Roadmap

See the [open issues](https://github.com/SynthRAD2025/algorithm-template/issues) for a list of proposed features (and known issues).

<!-- CONTRIBUTING -->
## Contributing

Contributions are what makes the open-source community such an amazing place to learn, inspire, and create.
Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<!-- LICENSE -->
## License

Distributed under the GNU General Public License v3.0. See `LICENSE.md` for more information.

<!-- CONTACT -->
## Contact

Matteo Maspero - [@matteomasperonl](https://bsky.app/profile/matteomaspero.bsky.social) - m.maspero@umcutrecht.nl

Project Link: [https://github.com/SynthRAD2025/algorithm-template](https://github.com/SynthRAD2025/algorithm-template)


<!-- ACKNOWLEDGEMENTS 
## Acknowledgements

* []()
* []()
* []()
-->

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/
#reference-style-links -->
[contributors-shield]: https://img.shields.io/github/contributors/SynthRAD2025/algorithm-template.svg?style=flat-square
[contributors-url]: https://github.com/SynthRAD2025/algorithm-template/graphs/contributors
[forks-shield]: https://img.shields.io/github/forks/SynthRAD2025/algorithm-template.svg?style=flat-square
[forks-url]: https://github.com/SynthRAD2025/algorithm-template/network/members
[stars-shield]: https://img.shields.io/github/stars/SynthRAD2025/algorithm-template.svg?style=flat-square
[stars-url]: https://github.com/SynthRAD2025/algorithm-template/stargazers
[issues-shield]: https://img.shields.io/github/issues/SynthRAD2025/algorithm-template.svg?style=flat-square
[issues-url]: https://github.com/SynthRAD2025/algorithm-template/issues
[license-shield]: https://img.shields.io/github/license/SynthRAD2025/algorithm-template.svg?style=flat-square
[license-url]: https://github.com/SynthRAD2025/algorithm-template/blob/master/LICENSE.md
