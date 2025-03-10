# Inference Instructions

Briefly, you need to:

1. Download a checkpoint.
2. Get the code.
3. Load the checkpoint.
4. Get activations.

Details are below.

## Download a Checkpoint

First, download an SAE checkpoint from the Huggingface collection (will be made public after review).

For instance, you can choose the SAE trained on OpenAI's CLIP ViT-B/16 with ImageNet-1K activations.

You can use `wget` if you want:

```sh
wget <REDACTED>
```

## Get the Code

The easiest way to do this is to clone the code:

```
git clone https://github.com/anonymous-saev/anonymous-saev
```

You can also install the package from git if you use uv (not sure about pip or cuda):

```sh
uv add git+https://github.com/anonymous-saev/anonymous-saev
```

Or clone it and install it as an editable with pip, lik `pip install -e .` in your virtual environment.

Then you can do things like `from saev import ...`.

.. note::
  If you struggle to get `saev` installed, open an issue on [GitHub](https://github.com/anonymous-saev/anonymous-saev) and I will figure out how to make it easier.

## Load the Checkpoint

```py
import saev.nn

sae = saev.nn.load("PATH_TO_YOUR_SAE_CKPT.pt")
```

Now you have a pretrained SAE.

## Get Activations

This is the hardest part.
We need to:

1. Pass an image into a ViT
2. Record the dense ViT activations at the same layer that the SAE was trained on.
3. Pass the activations into the SAE to get sparse activations.
4. Do something interesting with the sparse SAE activations.

Below is example code to do it using the `saev` package.

```py
import saev.nn
import saev.activations

img_transform = saev.activations.make_img_transform("clip", "ViT-B-16/openai")

vit = saev.activations.make_vit("clip", "ViT-B-16/openai")
recorded_vit = saev.activations.RecordedVisionTransformer(vit, 196, True, [10])

img = Image.open("example.jpg")

x = img_transform(img)
# Add a batch dimension
x = x[None, ...]
_, vit_acts = recorded_vit(x)
# Re-select the only element in the batch, and ignore the CLS token.
vit_acts = vit_acts[0, 1:, :]

x_hat, f_x, loss = sae(vit_acts)
```

Now you have the reconstructed x (`x_hat`) and the sparse representation of all patches in the image (`f_x`).

You might select the dimensions with maximal values for each patch and see what other images are maximimally activating.

.. todo::
  Provide documentation for how get maximally activating images.

