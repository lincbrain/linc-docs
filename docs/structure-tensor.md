# Calculate Structure Tensors

## Prerequisites

Ensure the repository is cloned and its dependencies are installed.

## Steps to Calculate Structure Tensor

1. Run Structure Tensor Calculation

Use the process_zarr function to compute the structure tensor from your input Zarr file. Adjust sigma and rho to control the smoothing parameters:
```
input = "/scratch/input.zarr"
output = "/scratch/output.zarr"
process_zarr(input, output, sigma=0, rho=5)
```
	•	sigma: Controls the Gaussian smoothing applied before computing derivatives.
	•	rho: Controls the Gaussian smoothing applied after computing the tensor.

2. Generate Pyramid for Multi-Resolution Data

To enable visualization or efficient access, create a resolution pyramid with generate_pyramid:
`
generate_pyramid(Zarr.zopen(output, "w"), levels=6, mode="mean")

	•	levels: Specifies the number of pyramid levels.
	•	mode: Determines the aggregation method (e.g., mean).
`
3. Add OME Metadata

To make the output Zarr file compatible with tools like Neuroglancer or Napari, write OME-compliant metadata:
`
write_ome_metadata(Zarr.zopen(output, "w"), ["c", "z", "y", "x"])

	•	Dimensions: ["c", "z", "y", "x"] corresponds to the channel, depth, height, and width dimensions of the data.
`
Example Workflow
	1.	Input: /scratch/input.zarr
	2.	Output: /scratch/output.zarr
`
process_zarr("/scratch/input.zarr", "/scratch/output.zarr", sigma=0, rho=5)
generate_pyramid(Zarr.zopen("/scratch/output.zarr", "w"), levels=6, mode="mean")
write_ome_metadata(Zarr.zopen("/scratch/output.zarr", "w"), ["c", "z", "y", "x"])
`
This pipeline will produce a multi-resolution Zarr dataset ready for visualization and further analysis.
