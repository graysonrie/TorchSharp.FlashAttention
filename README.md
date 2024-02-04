# TorchSharp.FlashAttention

## Introduction
TorchSharp.FlashAttention is a C# wrapper for the Flash Attention algorithm, leveraging the capabilities of TorchSharp for efficient deep learning in .NET environments. The Flash Attention algorithm, developed by Dao-AILab, is a groundbreaking method for accelerating attention computation in Transformer models. It significantly reduces memory usage and computation time, enabling faster and more efficient processing of large-scale data, especially in natural language processing and computer vision tasks.

## Installation from NuGet

TorchSharp.FlashAttention is available on NuGet. Due to the size of the binaries, we split it into two packages for windows (`TorchSharp.FlashAttention-windows`) and linux (`TorchSharp.FlashAttention-linux`). The packages already come with the precompiled binaries, so you don't need to install anything (except the preprequisites).

### Prerequisites

- .NET SDK 6.0+
- `TorchSharp-cuda-windows` or `TorchSharp-cuda-linux` package, version 0.101.6+

For building from source, see [below](#building-from-source).

## Usage

All the attention-related functions in the flash_attn package have been ported over, and can be used. The rest of the flash-attention operations (like FusedDense, LayerNorm, etc.) are going to be added into future versions. 

For each function, we allow all the parameters that are accessible through the Python interface.

The package currently references FlashAttention 2.5.2, including AliBi embeddings and forward with KV cache. 

The interfaces that have been ported over:

```cs
// FlashAttentionInterface
FlashAttentionInterface.flash_attn_func(...);
FlashAttentionInterface.flash_attn_kvpacked_func(...);
FlashAttentionInterface.flash_attn_qkvpacked_func(...);
FlashAttentionInterface.flash_attn_varlen_func(...);
FlashAttentionInterface.flash_attn_varlen_kvpacked_func(...);
FlashAttentionInterface.flash_attn_varlen_qkvpacked_func(...);
FlashAttentionInterface.flash_attn_with_kvcache(...);

// BertPadding
BertPadding.pad_input(...);
BertPadding.unpad_input(...);
BertPadding.unpad_input_for_concatenated_sequences(...);
```


### Example using the interface

Here is a simple example for using the FlashAttention interface in C#:

```cs
using TorchSharp.FlashAttention;

var (batch_size, seqlen, headdim, nheads) = (5, 12, 32, 4)
var qkv = torch.rand([batch_size, seqlen, 3, nheads, headdim]).half().cuda();
var (result, _, _) = FlashAttentionInterface.flash_attn_qkvpacked_func(qkv);
```

Comparison to Python:

```python
from flash_attn import flash_attn_qkvpacked_func

batch_size, seqlen, headdim, heads = 5, 12, 32, 4
qkv = torch.rand(batch_size, seqlen, 3, heads, headdim).half().cuda()
res = flash_attn_qkvpacked_func(qkv)
```

### Example using the FlashAttention module

In addition to the interface, we also include a custom TorchSharp module for applying QKV packed attention, with a custom key-padding mask:

```cs
using TorchSharp.FlashAttention;
using TorchSharp;


class MyModule<torch.Tensor, torch.Tensor, torch.Tensor> {
    private FlashAttention _flash;
    // ... rest of your fields

    public MyModule() : base("MyModule) {
        _flash = new(softmax_scale: 1, attention_dropout: 0.1, causal: true);
        // ... rest of your module

        RegisterComponents();
    }


    public override Tensor forward(Tensor input, Tensor key_padding_mask) {
        // ...
        var attnOutput = _flash.forward(input, key_padding_mask);
        // ...
    }
}

```

## Building from Source

There are multiple steps to building from source - we need to compile the cuda binaries, build the Native library bindings, and then build the C# library. 

Compiling the cuda binaries can take a long time, and therefore I included them as LFS objects in the repository. If you want to recompile, I include instructions below how to recompile. 

Step 1: Clone the repository:
   ```bash
   git clone https://github.com/shaltielshmid/TorchSharp.FlashAttention.git
   cd TorchSharp.FlashAttention
   git lfs pull
   ```

The next steps varies whether you are on Windows or Linux. 

On windows:

- Open Visual Studio (I tested it with VS2022)
- Build all in Release
    - The first time you do this, this can take some time because it is downloading and extracting the libtorch bindings, and cloning the FlashAttention repository and applying a patch to make it exportable for C++. 
- The built binaries can be found in:
    `...\TorchSharp.FlashAttention\TorchSharp.FlashAttention\obj\Debug\net6.0\`

On linux:

- Navigate to the directory
- Run:
    ```bash
    export CUDA_PATH_V12_1=/path/to/cuda/toolkit/root
    dotnet build TorchSharp.FlashAttention -c Release
    ```

### Compiling Flash Attention cuda binaries

- Make sure you run a build at least once, so that the source code is retrieved.
- Navigate to `TorchSharp.FlashAttention\Redist\flash-attn-2.5.2`
- Run either `compile_flash.bat` (for windows) or `bash compile_flash.sh` (for linux). 

## Acknowledgments
This project is a C# wrapper around the original [Flash Attention implementation by Dao-AILab](https://github.com/Dao-AILab/flash-attention). Immense gratitude goes to the creators and contributors of Flash Attention for their innovative work and for providing guidelines to encourage community-driven adaptations and extensions.

## Contributions
Contributions to TorchSharp.FlashAttention are warmly welcomed. Whether it's adding new features, improving documentation, or fixing bugs, your input is valuable. Please feel free to submit pull requests or open issues to discuss potential changes or report bugs.

