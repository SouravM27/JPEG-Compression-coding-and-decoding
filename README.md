# JPEG Compression and Decompression Notebook

This notebook implements a simplified JPEG compression and decompression algorithm, demonstrating the core concepts involved in converting an image to a compressed format and then reconstructing it.

## Table of Contents
1.  [Introduction](#introduction)
2.  [Core Components and Helper Functions](#core-components-and-helper-functions)
3.  [Compression Process](#compression-process)
4.  [Decompression Process](#decompression-process)
5.  [Usage](#usage)

## Introduction
This notebook provides a step-by-step implementation of JPEG (Joint Photographic Experts Group) compression and decompression. It covers essential stages such as color space conversion, chrominance subsampling, Discrete Cosine Transform (DCT), quantization, zigzag scanning, run-length encoding, and Huffman coding for compression, followed by the reverse process for decompression.

## Core Components and Helper Functions

The notebook defines several functions and data structures crucial for the JPEG algorithm:

*   **`zigzag(matrix: np.ndarray) -> np.ndarray`**: Converts an 8x8 matrix into a 1D array using a zigzag scan pattern, a method to group low-frequency coefficients together.
*   **`trim(array: np.ndarray) -> np.ndarray`**: Removes trailing zeros from a NumPy array, handling cases where the array becomes empty by adding a single zero (likely for DC component representation).
*   **`run_length_encoding(array: np.ndarray) -> list`**: Encodes a 1D array of zigzagged coefficients into a run-length encoded stream. It differentiates between DC components (represented by `(size, amplitude)`) and AC components (`(run_length, size, amplitude)`), and inserts `EOB` (End Of Block) markers.
*   **`get_freq_dict(array: list) -> dict`**: Calculates the frequency of each unique symbol in a list, forming the basis for Huffman coding.
*   **`lowest_prob_pair(p: dict)`**: A helper function for `find_huffman` that identifies the two symbols with the lowest probabilities in a frequency distribution.
*   **`find_huffman(p: dict) -> dict`**: Generates a Huffman code for a given frequency distribution `p`, assigning shorter codes to more frequent symbols.
*   **`rgb_to_ycbcr(image_bgr: np.ndarray) -> np.ndarray`**: Converts an image from BGR color space (OpenCV's default) to YCbCr color space using the BT.601 standard. This separates luminance (Y) from chrominance (Cb, Cr) for more efficient compression.
*   **`QTY` and `QTC`**: Standard quantization tables for luminance (Y) and chrominance (Cb, Cr) channels, respectively. These tables determine the level of detail retained after quantization.

## Compression Process

The main compression logic (`8FMibr0oZzDJ`) performs the following steps:

1.  **Load Image & Color Space Conversion**: Reads an input image (`pepper.bmp`) and converts it from BGR to YCbCr using `rgb_to_ycbcr`.
2.  **Level Shifting**: Subtracts 128 from each pixel value in the Y, Cb, and Cr channels to center the data around zero, which is beneficial for DCT.
3.  **Chrominance Subsampling**: Applies 4:2:2 subsampling to the Cb and Cr channels. This reduces the chrominance data by averaging 2x2 blocks, leveraging the human eye's lower sensitivity to color detail.
4.  **Padding**: Pads the Y, Cb, and Cr channels to ensure their dimensions are multiples of the `windowSize` (8x8 blocks), which is necessary for block-based processing.
5.  **Discrete Cosine Transform (DCT)**: Applies DCT to 8x8 blocks of each padded channel. DCT transforms spatial domain pixel data into frequency domain coefficients.
6.  **Quantization**: Divides the DCT coefficients by the respective quantization tables (`QTY` for Y, `QTC` for Cb/Cr) and rounds the results. This is a lossy step that discards less perceptually significant information.
7.  **Zigzag Scan**: Converts the quantized 8x8 blocks into 1D arrays using the `zigzag` function, arranging coefficients in increasing order of frequency.
8.  **Run-Length Encoding (RLE)**: Applies `run_length_encoding` to the zigzagged coefficients to represent sequences of zeros efficiently.
9.  **Huffman Coding**: Generates frequency tables using `get_freq_dict` from the RLE output and then creates Huffman codes using `find_huffman` to assign variable-length codes to the RLE symbols, achieving entropy coding (lossless compression).
10. **Bitstream Generation & Saving**: Concatenates the Huffman-encoded bitstreams for Y, Cb, and Cr channels and saves them to a file named `CompressedImage.bin` (or `CompressedImage.asfh` in the decompression step).
11. **Compression Ratio Calculation**: Calculates and prints the compression ratio achieved.

## Decompression Process

The decompression logic (`new_cell_4`) reverses the compression steps:

1.  **Read Bitstreams**: Reads the Huffman-encoded bitstreams for Y, Cb, and Cr from `CompressedImage.asfh`.
2.  **Build Reverse Huffman Maps**: Creates reverse lookup dictionaries from the Huffman encoding tables (`yHuffman`, `crHuffman`, `cbHuffman`) to map bit codes back to symbols.
3.  **Huffman Decoding**: Decodes the bitstreams using the reverse Huffman maps to retrieve the original run-length encoded symbols.
4.  **Inverse Run-Length Decoding**: Reconstructs the zigzagged coefficients from the run-length encoded symbols.
5.  **Inverse Zigzag Scan**: Reconstructs the 8x8 quantized blocks from their 1D zigzag arrays.
6.  **De-quantization**: Multiplies the quantized coefficients by their respective quantization tables (`QTY`, `QTC`) to approximate the original DCT coefficients.
7.  **Inverse Discrete Cosine Transform (IDCT)**: Applies IDCT to the de-quantized blocks to convert them back from the frequency domain to the spatial domain.
8.  **Remove Padding & Level Shift**: Removes any padding added during compression and adds 128 back to the pixel values to restore their original range.
9.  **Inverse Subsampling (Upsampling)**: Upsamples the Cb and Cr channels to the original image dimensions using linear interpolation (`cv2.resize`).
10. **YCbCr to BGR Conversion**: Stacks the Y, Cb, and Cr channels to form a YCbCr image and then converts it back to the BGR color space.
11. **Save Reconstructed Image**: Saves the final reconstructed image as `ReconstructedImage.jpg`.

## Usage
To use this notebook, ensure you have an image named `pepper.bmp` in the same directory as the notebook. Run all cells sequentially to perform the compression and decompression, and observe the compression ratio and the reconstructed image.