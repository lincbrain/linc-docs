# HiP-CT data

HiP-CT stands for hierarchical phase-contrast tomography.
For more information on the modality see the [HiP-CT website](https://mecheng.ucl.ac.uk/hip-ct/).
Most of the time HiP-CT consists of a single 'low' (but still quite high!) resolution scan of a whole sample, and then one or more 'high' resolution scans of particular volumes of interest within the same sample.

## Data acquisition and naming

There are HiP-CT scans of two donors, S45 and I58.
The data are part of the [Pilot human brains dataset](https://lincbrain.org/dandiset/000005) on lincbrain.

Datasets have the name `sub-<donor>_sample-<sample name>_chunk<chunk number>_PC.ome.zarr`.
The chunk number is set to `yyxx`, where yy is the resolution of the scan in micrometers, and xx is a scan number that is unique between datasets.
For example, a chunk number of `402` is the second 4um scan.
The lowest resolution dataset (equivalently, highest chunk number) is the overview dataset that contains an image of the whole sample.
There are then one or more datasets at higher resolution.

## Registration

Each high resolution dataset is registered to the overview dataset using [`hipct-reg`](https://github.com/hipCTProject/hipct-reg).
Each transform maps the pixel coordinates of the high resolution zoom datasets to the overview full-sample dataset.
The transform is of the form:

$$
T(\mathbf{x}) = s \left ( \mathbf{R} \cdot\mathbf{x} \right ) + \mathbf{t}
$$

Where

- $s$ is a scaling factor
- $\mathbf{R}$ is a rotation matrix that rotates about the z-axis.
- $\mathbf{t}$ is a translation vector

The rotation matrix takes the form

$$
\mathbf{R} = \begin{pmatrix}
\cos(-\theta) & \sin(-\theta) & 0 \\
-\sin(-\theta) & \cos(-\theta) & 0 \\
0 & 0 & 1 \\
\end{pmatrix}
$$

The transform has 5 parameters (1 scaling, 1 rotation, 3 translation).

### Registration accuracy

The HiP-CT registration pipeline is designed to be accurate enough to approximately visually align two datasets. **It is not designed for pixel perfect registration.** We recommend anyone doing analysis relying on the registration between two datasets performs a more accurate registration themselves, using our registration as an initial starting point.

### Registration data


| Dataset | Overview dataset | $\mathbf{t}$ | $s$ | $\theta$ (degrees) |
| -------- | -------- | -------- | -------- | -------- |
| `sub-I58_sample-blockIC2_chunk-101_PC`     | `sub-I58_sample-blockIC2_chunk-401_PC`     |   (3746.4467746089194, 3660.0610535229666, 1127.3616712249263)   | 0.2016106611944697 | -6.496401244495998 |
| `sub-I58_sample-blockIC2_chunk-201_PC`    | `sub-I58_sample-blockIC2_chunk-401_PC`     |   (2119.695989239648, 2321.6572801771194, 971.0041825720597)   | 0.5181803632402424 | 0.5004912711693281 |
