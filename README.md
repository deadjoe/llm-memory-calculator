# LLM Memory Calculator

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![HTML](https://img.shields.io/badge/HTML-5-orange)](https://developer.mozilla.org/en-US/docs/Web/HTML)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES6-blue)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Three.js](https://img.shields.io/badge/Three.js-r128-green)](https://threejs.org/)

A standalone HTML/JavaScript application for calculating GPU memory requirements for large language models (LLMs). This tool assists AI practitioners in determining hardware requirements for inference, fine-tuning, and training from scratch.

![LLM Memory Calculator Screenshot](https://raw.githubusercontent.com/deadjoe/llm-memory-calculator/main/screenshot.jpg)

## Features

- Calculate memory requirements for models ranging from 7B to 175B+ parameters
- Support for different precision levels (32-bit, 16-bit, 8-bit and 4-bit quantization)
- Estimates for different operation modes (inference, fine-tuning, training)
- Hardware recommendations for both NVIDIA GPUs and Apple Silicon
- Interactive 3D visualization of memory allocation components
- Single HTML file with no server requirements

## Usage

### Online Version

You can access the online version of this calculator at: https://deadjoe.github.io/llm-memory-calculator/

### Local Usage

Simply open `llm-memory-calculator.html` or `index.html` in any modern web browser. No installation or server setup required.

### Hosting Your Own Copy

To host this calculator on GitHub Pages:

1. Fork this repository
2. Go to your fork's Settings tab
3. Navigate to "Pages" in the left sidebar
4. Under "Build and deployment", select "Deploy from a branch"
5. Select "main" branch and "/ (root)" folder
6. Click "Save"
7. After a few minutes, your calculator will be available at `https://yourusername.github.io/llm-memory-calculator/`

## Memory Calculation Algorithm

The calculator uses a comprehensive algorithm to estimate memory requirements based on real-world usage patterns rather than theoretical minimums. Below is the detailed calculation methodology:

### Core Memory Components

1. **Base Model Size (GB)**:
   - Parameter Count (P) × Bytes per Parameter
   - Bytes per parameter varies by precision:
     - 32-bit (FP32): 4 bytes/parameter
     - 16-bit (FP16/BF16): 2 bytes/parameter
     - 8-bit Quantized: 1.2 bytes/parameter (includes quantization overhead)
     - 4-bit Quantized: 0.65 bytes/parameter (includes metadata overhead)

2. **Framework Overhead**:
   - 15% additional to base model size
   - Accounts for PyTorch/TensorFlow runtime memory allocation

3. **Operation Mode Multiplier**:
   - Inference: 1.05× multiplier (5% overhead)
   - Fine-tuning: 2.5× multiplier (150% overhead)
   - Training from Scratch: 4.0× multiplier (300% overhead)

4. **Inference-specific Components** (only applied in inference mode):
   - **KV Cache**:
     - KV Cache (GB) = (Num Layers × 2 × Hidden Size × Context Length × Bytes per Parameter) ÷ 10^9
     - Uses model-specific values for layers and hidden dimensions:
       - 7B models: 32 layers, 4096 hidden dimension
       - 13B models: 40 layers, 5120 hidden dimension
       - 34B models: 60 layers, 6656 hidden dimension
       - 70B models: 80 layers, 8192 hidden dimension
       - 175B+ models: 96 layers, 12288 hidden dimension
     - Uses 8192 as the default context length

   - **Activation Memory**:
     - Activation Memory (GB) = (Hidden Size × Context Length × Bytes per Parameter × 2) ÷ 10^9

5. **CUDA/Runtime Buffer**:
   - 8% of total memory allocation
   - Accounts for CUDA workspace and system buffers

6. **System Architecture Factor**:
   - PC/NVIDIA: 1.0× (standard)
   - Apple Silicon: 0.9× (unified memory efficiency)

7. **Hardware Recommendation Safety Margin**:
   - 10% additional buffer for recommendations
   - Ensures comfortable operation without memory pressure

### Complete Formula

For inference mode:

![equation](https://latex.codecogs.com/png.latex?%5Cdpi%7B120%7D%20%5Cbg_white%20%5Ctext%7BMemory%7D_%7B%5Ctext%7BGB%7D%7D%20%3D%20%5Cleft%5B%5Cleft%28%5Ctext%7BBase%20Model%20Size%7D%20%5Ctimes%201.15%20%5Ctimes%201.05%5Cright%29%20&plus;%20%5Ctext%7BKV%20Cache%7D%20&plus;%20%5Ctext%7BActivation%20Memory%7D%20&plus;%20%5Ctext%7BBuffer%7D%5Cright%5D%20%5Ctimes%20%5Ctext%7BSystem%20Factor%7D)

Where:

![equation](https://latex.codecogs.com/png.latex?%5Cdpi%7B120%7D%20%5Cbg_white%20%5Ctext%7BKV%20Cache%7D_%7B%5Ctext%7BGB%7D%7D%20%3D%20%5Cfrac%7B%5Ctext%7BNum%20Layers%7D%20%5Ctimes%202%20%5Ctimes%20%5Ctext%7BHidden%20Size%7D%20%5Ctimes%20%5Ctext%7BContext%20Length%7D%20%5Ctimes%20%5Ctext%7BBytes%20per%20Param%7D%7D%7B10%5E9%7D)

![equation](https://latex.codecogs.com/png.latex?%5Cdpi%7B120%7D%20%5Cbg_white%20%5Ctext%7BActivation%20Memory%7D_%7B%5Ctext%7BGB%7D%7D%20%3D%20%5Cfrac%7B%5Ctext%7BHidden%20Size%7D%20%5Ctimes%20%5Ctext%7BContext%20Length%7D%20%5Ctimes%20%5Ctext%7BBytes%20per%20Param%7D%20%5Ctimes%202%7D%7B10%5E9%7D)

![equation](https://latex.codecogs.com/png.latex?%5Cdpi%7B120%7D%20%5Cbg_white%20%5Ctext%7BBuffer%7D_%7B%5Ctext%7BGB%7D%7D%20%3D%20%5Ctext%7BTotal%20Memory%20Before%20Buffer%7D%20%5Ctimes%200.08)

For training/fine-tuning:

![equation](https://latex.codecogs.com/png.latex?%5Cdpi%7B120%7D%20%5Cbg_white%20%5Ctext%7BMemory%7D_%7B%5Ctext%7BGB%7D%7D%20%3D%20%5Cleft%5B%5Cleft%28%5Ctext%7BBase%20Model%20Size%7D%20%5Ctimes%201.15%20%5Ctimes%20%5Ctext%7BMode%20Factor%7D%5Cright%29%20&plus;%20%5Ctext%7BBuffer%7D%5Cright%5D%20%5Ctimes%20%5Ctext%7BSystem%20Factor%7D)

### Hardware Recommendations

The calculator accounts for actual usable VRAM in its recommendations by reserving:
- ~2-8GB for system/CUDA runtime on NVIDIA GPUs
- ~2-20GB for system needs on Apple Silicon (depending on total RAM)

## Development

The calculator is built with vanilla JavaScript and Three.js for visualizations. All computation happens client-side with no external API dependencies.

### Technologies Used

- HTML5 and CSS3 for layout and styling
- Vanilla JavaScript (ES6+) for calculations
- Three.js for 3D memory visualization
- KaTeX for formula rendering
- Tween.js for smooth animations

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Developed based on practical experience and benchmarks from running large language models
- Memory estimation formulas derived from real-world LLM deployment scenarios