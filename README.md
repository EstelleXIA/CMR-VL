# Construction of VLM for CMR Interpretation

This is the Github Repository for "Vision-Language Model for Automated Cardiac MRI Interpretation Enhances Efficiency in Acute Myocardial Infarction Assessment". We have released our trained model on [HuggingFace](https://huggingface.co/EstelleXIA/CMR-VL).

## Installation

The project relies largely on [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory). You can refer to the installation guide in their repository. Or you can use the environment configuration file located in `./environment.yml`.

## Training

### Cardiac Functional Parameters

We selected two open-source datasets, [M&Ms](https://www.ub.edu/mnms/) and [ACDC](https://www.kaggle.com/datasets/anhoangvo/acdc-dataset), which include a total of 934 images, to build 3D segmentation models for the left ventricle, right ventricle, and myocardium. The model uses the standard [nnU-Net](https://github.com/MIC-DKFZ/nnUNet) configuration. The training was conducted using 1×NVIDIA A100 (40GB) GPU.

### VLM-based Radiological Findings Generation

Data preparation as well as model training should follow the workflow of [LLaMA-Factory](https://github.com/hiyouga/LLaMA-Factory) . The following provides the details.

#### Visual-Question Answering Pairs

The VQA data should be organized as a List of Dict, with each Dict representing one VQA sample. For example, an effusion identification sample should be structured as follows:

```
{
    "messages": 
    [
        {"role": "user",
         "content": "<image><image><image><image><image><image><image><image><image><image>Is there any pericardial effusion present?"},
        {"role": "assistant",
         "content": "There is a small amount of pericardial effusion."}
         ],
    "images": ["patient_name/effusion_0.png", "patient_name/effusion_1.png", "patient_name/effusion_2.png", "patient_name/effusion_3.png", "patient_name/effusion_4.png", "patient_name/effusion_5.png", "patient_name/effusion_6.png", "patient_name/effusion_7.png", "patient_name/effusion_8.png", "patient_name/effusion_9.png"],
    "videos": []
    }
```

A VQA sample for motion analysis should be structured like this:

```
{
    "messages": 
    [
        {"role": "user",
         "content": "<video>Is the wall motion of the ventricles coordinated? If abnormalities in wall motion are observed in various segments, report the abnormal segments and the abnormal motion patterns."},
        {"role": "assistant",
         "content": "The left ventricular apex shows thinning, with decreased wall motion and significantly reduced contractile function, and there is paradoxical motion at the apex."}
         ],
    "images": [],
    "videos": ["patient_name/sa-cine_crop.mp4"]
}
```

Next, store the constructed VQA pairs in a JSON file so that they can be included in `dataset_info.json`, which records all VQA information for training and validation.

A structured `dataset_info.json` would look roughly like this:

```
{
  "cmr_train_sharegpt": {
    "file_name": "cmr_train.json",
    "formatting": "sharegpt",
    "columns": {
      "messages": "messages",
      "videos": "videos",
      "images": "images"
    },
    "tags": {
      "role_tag": "role",
      "content_tag": "content",
      "user_tag": "user",
      "assistant_tag": "assistant"
    }
  },
  "cmr_val_sharegpt": {
    "file_name": "cmr_val.json",
    "formatting": "sharegpt",
    "columns": {
      "messages": "messages",
      "videos": "videos",
      "images": "images"
    },
    "tags": {
      "role_tag": "role",
      "content_tag": "content",
      "user_tag": "user",
      "assistant_tag": "assistant"
    }
  },
  "cmr_test_sharegpt": {
    "file_name": "cmr_test.json",
    "formatting": "sharegpt",
    "columns": {
      "messages": "messages",
      "videos": "videos",
      "images": "images"
    },
    "tags": {
      "role_tag": "role",
      "content_tag": "content",
      "user_tag": "user",
      "assistant_tag": "assistant"
    }
  },
}
```

#### Model Training

To start model training, a configuration file documenting all training details is required. The configuration file for our model training is provided at `./qwen2_5vl_sft_train.yaml`.

Then, run the following command to start the training. 

```
llamafactory-cli train qwen2_5vl_sft_train.yaml
```

The training took about 10 hours on 4×NVIDIA A800 (80GB) GPUs.

#### Inference

After model training, run the following command for inference.The configuration file is provided at `./qwen2_5vl_sft_val.yaml`.

```
llamafactory-cli train qwen2_5vl_sft_val.yaml
```



## Terms of use

The code should only be used for academic research purposes.



