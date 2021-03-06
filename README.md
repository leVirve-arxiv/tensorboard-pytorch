# tensorboard-pytorch

[![Build Status](https://travis-ci.org/lanpa/tensorboard-pytorch.svg?branch=master)](https://travis-ci.org/lanpa/tensorboard-pytorch)
[![PyPI version](https://badge.fury.io/py/tensorboardX.svg)](https://badge.fury.io/py/tensorboardX)
[![Downloads](https://img.shields.io/badge/pip--downloads-5K+-brightgreen.svg)](https://bigquery.cloud.google.com/savedquery/966219917372:edb59a0d70c54eb687ab2a9417a778ee)

Visualize your training events with TensorBoard for PyTorch and Chainer.

* Support `PyTorch`, `Chainer`
* Support `scalar`, `image`, `histogram`, `audio`, `text`, `graph`, `onnx_graph`, `embedding` and `pr_curve` summaries.

## Demo

[demo site](http://35.197.26.245:6006) (TensorBoard for `demo.py` and some images generated by BEGAN)

*If the demo code `onnx_graph.py` is not working, you have to build pytorch and onnx from source.*

## Requirement

* latest anaconda2 / anaconda3
* PyTorch 0.3
* torchvision 0.2
* tensorflow (for tensorboard web server)

## Install

`pip install tensorboardX`

or build from source:

`pip install git+https://github.com/lanpa/tensorboard-pytorch`

## API

[Documents](http://tensorboard-pytorch.readthedocs.io/en/latest/tensorboard.html)

## Example

* Run the demo script: `python demo.py`
* Use TensorBoard with `tensorboard --logdir runs`

```python
# demo.py
import torch
import torchvision.utils as vutils
import numpy as np
import torchvision.models as models
from torchvision import datasets
from tensorboardX import SummaryWriter

resnet18 = models.resnet18(False)
writer = SummaryWriter()
sample_rate = 44100
freqs = [262, 294, 330, 349, 392, 440, 440, 440, 440, 440, 440]

for n_iter in range(100):

    ''' scalar '''
    dummy_s1 = torch.rand(1)
    dummy_s2 = torch.rand(1)
    # data grouping by `slash`
    writer.add_scalar('data/scalar1', dummy_s1[0], n_iter)
    writer.add_scalar('data/scalar2', dummy_s2[0], n_iter)

    ''' multiple scalars '''
    writer.add_scalars('data/scalar_group', {'xsinx': n_iter * np.sin(n_iter),
                                             'xcosx': n_iter * np.cos(n_iter),
                                             'arctanx': np.arctan(n_iter)}, n_iter)

    dummy_img = torch.rand(32, 3, 64, 64)  # output from network
    if n_iter % 10 == 0:
        ''' image '''
        x = vutils.make_grid(dummy_img, normalize=True, scale_each=True)
        writer.add_image('Image', x, n_iter)

        ''' audio '''
        dummy_audio = torch.zeros(sample_rate * 2)
        for i in range(x.size(0)):
            # amplitude of sound should in [-1, 1]
            dummy_audio[i] = np.cos(freqs[n_iter // 10] * np.pi * float(i) / float(sample_rate))
        writer.add_audio('myAudio', dummy_audio, n_iter, sample_rate=sample_rate)

        ''' text '''
        writer.add_text('Text', 'text logged at step:' + str(n_iter), n_iter)

        ''' histogram '''
        for name, param in resnet18.named_parameters():
            writer.add_histogram(name, param.clone().cpu().data.numpy(), n_iter)

        ''' pr_curve '''
        # needs tensorboard 0.4RC or later
        writer.add_pr_curve('xoxo', np.random.randint(2, size=100), np.random.rand(100), n_iter)

dataset = datasets.MNIST('mnist', train=False, download=True)
images = dataset.test_data[:100].float()
label = dataset.test_labels[:100]

''' embedding '''
features = images.view(100, 784)
writer.add_embedding(features, metadata=label, label_img=images.unsqueeze(1))

# export scalar data to JSON for external processing
writer.export_scalars_to_json("./all_scalars.json")
writer.close()
```

## Screenshots

<img src="screenshots/Demo.gif">

## Tweaks

To show more images in TensorBoard's image tab, just
modify the hardcoded `event_accumulator` in
`~/anaconda3/lib/python3.6/site-packages/tensorflow/tensorboard/backend/application.py`
as you wish.

For `tensorflow-tensorboard` > 0.17 see https://github.com/lanpa/tensorboard-pytorch/issues/44

## Reference

* [TeamHG-Memex/tensorboard_logger](https://github.com/TeamHG-Memex/tensorboard_logger)
* [dmlc/tensorboard](https://github.com/dmlc/tensorboard)
