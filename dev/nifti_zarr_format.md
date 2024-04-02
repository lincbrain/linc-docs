# NIfTI-Zarr

Neuroglancer can currently load two volume formats that we are interested in:
- [OME-Zarr](https://ngff.openmicroscopy.org/0.4/index.html) is developed be the Open Microscopy Environment (OME) 
  and targets a diverse range of microscopy applications. It leverages the 
  [Zarr v2 specification](https://zarr.readthedocs.io/en/stable/spec/v2.html), and consists of a 
  [zarr group](https://zarr.readthedocs.io/en/stable/spec/v2.html#groups) of 
  [zarr arrays](https://zarr.readthedocs.io/en/stable/spec/v2.html#arrays) numbered `0` to `N`. 
  Each array contains a representation of the same data but at different resolutions, with `0` containing the
  highest resolution. It also stores [`"multiscales"` metadata](https://ngff.openmicroscopy.org/0.4/index.html#multiscale-md)
  in the root `.zattrs` dictionary that describe the arrays' axes, along with their relative scaling and (eventually) translations.
- [NIfTI](https://nifti.nimh.nih.gov/pub/dist/src/niftilib/nifti1.h) is widely used in the neuroimaging community. It is a single file
  binary format that contains a header section followed by the raw array. The whole thing can then be optionally gzipped. What's quite
  important to us is that the header contains a coordinate transform that converts voxel coordinates into a common "world" space.
  There are actually up to two transforms stored in the header (which is a source of massive confusion in the community):
  1. The "qform" is a pure rotation + translation encoded by quaternions;
  2. The "sform" is a general affine tranform encoded by a 3x4 matrix.
 
We would like neuroglancer to further handle:
- [NIfTI-Zarr](https://github.com/neuroscales/nifti-zarr/tree/draft/0.2) is a proposed format that simply adds the header section of
  the nifti format inside the ome-zarr hierarchy.

Since any nifti-zarr is a valid ome-zarr, the current behaviour of neuroglancer when given a nifti-zarr asset with the `zarr://` protocol 
is to only look at the OME metadata (that is, the scaling and translation along each axis). 
We would like to add a new `niftizarr://` protocol that instead uses the "sform" (or "qform", depending on their intent code) 
of the nifti header to map the array axes to some world space, like what's currently done when using the `nifti://` protocol.

Corresponding neuroglancer code:
- [OME-Zarr datasource](https://github.com/google/neuroglancer/tree/master/src/datasource/zarr)
- [NIfTI datasource](https://github.com/google/neuroglancer/tree/master/src/datasource/nifti)

## NIfTI

I am just copying useful sections from the NIfTI spec.

This is the content of the header portion:
```
struct nifti_1_header { /* NIFTI-1 usage         */  /* ANALYZE 7.5 field(s) */
                        /*************************/  /************************/

                                           /*--- was header_key substruct ---*/
 int   sizeof_hdr;    /*!< MUST be 348           */  /* int sizeof_hdr;      */
 char  data_type[10]; /*!< ++UNUSED++            */  /* char data_type[10];  */
 char  db_name[18];   /*!< ++UNUSED++            */  /* char db_name[18];    */
 int   extents;       /*!< ++UNUSED++            */  /* int extents;         */
 short session_error; /*!< ++UNUSED++            */  /* short session_error; */
 char  regular;       /*!< ++UNUSED++            */  /* char regular;        */
 char  dim_info;      /*!< MRI slice ordering.   */  /* char hkey_un0;       */

                                      /*--- was image_dimension substruct ---*/
 short dim[8];        /*!< Data array dimensions.*/  /* short dim[8];        */
 float intent_p1 ;    /*!< 1st intent parameter. */  /* short unused8;       */
                                                     /* short unused9;       */
 float intent_p2 ;    /*!< 2nd intent parameter. */  /* short unused10;      */
                                                     /* short unused11;      */
 float intent_p3 ;    /*!< 3rd intent parameter. */  /* short unused12;      */
                                                     /* short unused13;      */
 short intent_code ;  /*!< NIFTI_INTENT_* code.  */  /* short unused14;      */
 short datatype;      /*!< Defines data type!    */  /* short datatype;      */
 short bitpix;        /*!< Number bits/voxel.    */  /* short bitpix;        */
 short slice_start;   /*!< First slice index.    */  /* short dim_un0;       */
 float pixdim[8];     /*!< Grid spacings.        */  /* float pixdim[8];     */
 float vox_offset;    /*!< Offset into .nii file */  /* float vox_offset;    */
 float scl_slope ;    /*!< Data scaling: slope.  */  /* float funused1;      */
 float scl_inter ;    /*!< Data scaling: offset. */  /* float funused2;      */
 short slice_end;     /*!< Last slice index.     */  /* float funused3;      */
 char  slice_code ;   /*!< Slice timing order.   */
 char  xyzt_units ;   /*!< Units of pixdim[1..4] */
 float cal_max;       /*!< Max display intensity */  /* float cal_max;       */
 float cal_min;       /*!< Min display intensity */  /* float cal_min;       */
 float slice_duration;/*!< Time for 1 slice.     */  /* float compressed;    */
 float toffset;       /*!< Time axis shift.      */  /* float verified;      */
 int   glmax;         /*!< ++UNUSED++            */  /* int glmax;           */
 int   glmin;         /*!< ++UNUSED++            */  /* int glmin;           */

                                         /*--- was data_history substruct ---*/
 char  descrip[80];   /*!< any text you like.    */  /* char descrip[80];    */
 char  aux_file[24];  /*!< auxiliary filename.   */  /* char aux_file[24];   */

 short qform_code ;   /*!< NIFTI_XFORM_* code.   */  /*-- all ANALYZE 7.5 ---*/
 short sform_code ;   /*!< NIFTI_XFORM_* code.   */  /*   fields below here  */
                                                     /*   are replaced       */
 float quatern_b ;    /*!< Quaternion b param.   */
 float quatern_c ;    /*!< Quaternion c param.   */
 float quatern_d ;    /*!< Quaternion d param.   */
 float qoffset_x ;    /*!< Quaternion x shift.   */
 float qoffset_y ;    /*!< Quaternion y shift.   */
 float qoffset_z ;    /*!< Quaternion z shift.   */

 float srow_x[4] ;    /*!< 1st row affine transform.   */
 float srow_y[4] ;    /*!< 2nd row affine transform.   */
 float srow_z[4] ;    /*!< 3rd row affine transform.   */

 char intent_name[16];/*!< 'name' or meaning of data.  */

 char magic[4] ;      /*!< MUST be "ni1\0" or "n+1\0". */

} ;                   /**** 348 bytes total ****/
```

This is how axis scaling is stored (which corresponds to the OME multiscales metadata):
```
 DATA DIMENSIONALITY (as in ANALYZE 7.5):
 ---------------------------------------
   dim[0] = number of dimensions;
            - if dim[0] is outside range 1..7, then the header information
              needs to be byte swapped appropriately
            - ANALYZE supports dim[0] up to 7, but NIFTI-1 reserves
              dimensions 1,2,3 for space (x,y,z), 4 for time (t), and
              5,6,7 for anything else needed.

   dim[i] = length of dimension #i, for i=1..dim[0]  (must be positive)
            - also see the discussion of intent_code, far below

   pixdim[i] = voxel width along dimension #i, i=1..dim[0] (positive)
               - cf. ORIENTATION section below for use of pixdim[0]
               - the units of pixdim can be specified with the xyzt_units
                 field (also described far below).

 Number of bits per voxel value is in bitpix, which MUST correspond with
 the datatype field.  The total number of bytes in the image data is
   dim[1] * ... * dim[dim[0]] * bitpix / 8

 In NIFTI-1 files, dimensions 1,2,3 are for space, dimension 4 is for time,
 and dimension 5 is for storing multiple values at each spatiotemporal
 voxel.  Some examples:
   - A typical whole-brain FMRI experiment's time series:
      - dim[0] = 4
      - dim[1] = 64   pixdim[1] = 3.75 xyzt_units =  NIFTI_UNITS_MM
      - dim[2] = 64   pixdim[2] = 3.75             | NIFTI_UNITS_SEC
      - dim[3] = 20   pixdim[3] = 5.0
      - dim[4] = 120  pixdim[4] = 2.0
   - A typical T1-weighted anatomical volume:
      - dim[0] = 3
      - dim[1] = 256  pixdim[1] = 1.0  xyzt_units = NIFTI_UNITS_MM
      - dim[2] = 256  pixdim[2] = 1.0
      - dim[3] = 128  pixdim[3] = 1.1
   - A single slice EPI time series:
      - dim[0] = 4
      - dim[1] = 64   pixdim[1] = 3.75 xyzt_units =  NIFTI_UNITS_MM
      - dim[2] = 64   pixdim[2] = 3.75             | NIFTI_UNITS_SEC
      - dim[3] = 1    pixdim[3] = 5.0
      - dim[4] = 1200 pixdim[4] = 0.2
   - A 3-vector stored at each point in a 3D volume:
      - dim[0] = 5
      - dim[1] = 256  pixdim[1] = 1.0  xyzt_units = NIFTI_UNITS_MM
      - dim[2] = 256  pixdim[2] = 1.0
      - dim[3] = 128  pixdim[3] = 1.1
      - dim[4] = 1    pixdim[4] = 0.0
      - dim[5] = 3                     intent_code = NIFTI_INTENT_VECTOR
   - A single time series with a 3x3 matrix at each point:
      - dim[0] = 5
      - dim[1] = 1                     xyzt_units = NIFTI_UNITS_SEC
      - dim[2] = 1
      - dim[3] = 1
      - dim[4] = 1200 pixdim[4] = 0.2
      - dim[5] = 9                     intent_code = NIFTI_INTENT_GENMATRIX
      - intent_p1 = intent_p2 = 3.0    (indicates matrix dimensions)
```

This is how the more advanced "voxel to world" mapping is stored:
```
3D IMAGE (VOLUME) ORIENTATION AND LOCATION IN SPACE:
 ---------------------------------------------------
 There are 3 different methods by which continuous coordinates can
 attached to voxels.  The discussion below emphasizes 3D volumes, and
 the continuous coordinates are referred to as (x,y,z).  The voxel
 index coordinates (i.e., the array indexes) are referred to as (i,j,k),
 with valid ranges:
   i = 0 .. dim[1]-1
   j = 0 .. dim[2]-1  (if dim[0] >= 2)
   k = 0 .. dim[3]-1  (if dim[0] >= 3)
 The (x,y,z) coordinates refer to the CENTER of a voxel.  In methods
 2 and 3, the (x,y,z) axes refer to a subject-based coordinate system,
 with
   +x = Right  +y = Anterior  +z = Superior.
 This is a right-handed coordinate system.  However, the exact direction
 these axes point with respect to the subject depends on qform_code
 (Method 2) and sform_code (Method 3).

 N.B.: The i index varies most rapidly, j index next, k index slowest.
  Thus, voxel (i,j,k) is stored starting at location
    (i + j*dim[1] + k*dim[1]*dim[2]) * (bitpix/8)
  into the dataset array.

 N.B.: The ANALYZE 7.5 coordinate system is
    +x = Left  +y = Anterior  +z = Superior
  which is a left-handed coordinate system.  This backwardness is
  too difficult to tolerate, so this NIFTI-1 standard specifies the
  coordinate order which is most common in functional neuroimaging.

 N.B.: The 3 methods below all give the locations of the voxel centers
  in the (x,y,z) coordinate system.  In many cases, programs will wish
  to display image data on some other grid.  In such a case, the program
  will need to convert its desired (x,y,z) values into (i,j,k) values
  in order to extract (or interpolate) the image data.  This operation
  would be done with the inverse transformation to those described below.

 N.B.: Method 2 uses a factor 'qfac' which is either -1 or 1; qfac is
  stored in the otherwise unused pixdim[0].  If pixdim[0]=0.0 (which
  should not occur), we take qfac=1.  Of course, pixdim[0] is only used
  when reading a NIFTI-1 header, not when reading an ANALYZE 7.5 header.

 N.B.: The units of (x,y,z) can be specified using the xyzt_units field.

 METHOD 1 (the "old" way, used only when qform_code = 0):
 -------------------------------------------------------
 The coordinate mapping from (i,j,k) to (x,y,z) is the ANALYZE
 7.5 way.  This is a simple scaling relationship:

   x = pixdim[1] * i
   y = pixdim[2] * j
   z = pixdim[3] * k

 No particular spatial orientation is attached to these (x,y,z)
 coordinates.  (NIFTI-1 does not have the ANALYZE 7.5 orient field,
 which is not general and is often not set properly.)  This method
 is not recommended, and is present mainly for compatibility with
 ANALYZE 7.5 files.

 METHOD 2 (used when qform_code > 0, which should be the "normal" case):
 ---------------------------------------------------------------------
 The (x,y,z) coordinates are given by the pixdim[] scales, a rotation
 matrix, and a shift.  This method is intended to represent
 "scanner-anatomical" coordinates, which are often embedded in the
 image header (e.g., DICOM fields (0020,0032), (0020,0037), (0028,0030),
 and (0018,0050)), and represent the nominal orientation and location of
 the data.  This method can also be used to represent "aligned"
 coordinates, which would typically result from some post-acquisition
 alignment of the volume to a standard orientation (e.g., the same
 subject on another day, or a rigid rotation to true anatomical
 orientation from the tilted position of the subject in the scanner).
 The formula for (x,y,z) in terms of header parameters and (i,j,k) is:

   [ x ]   [ R11 R12 R13 ] [        pixdim[1] * i ]   [ qoffset_x ]
   [ y ] = [ R21 R22 R23 ] [        pixdim[2] * j ] + [ qoffset_y ]
   [ z ]   [ R31 R32 R33 ] [ qfac * pixdim[3] * k ]   [ qoffset_z ]

 The qoffset_* shifts are in the NIFTI-1 header.  Note that the center
 of the (i,j,k)=(0,0,0) voxel (first value in the dataset array) is
 just (x,y,z)=(qoffset_x,qoffset_y,qoffset_z).

 The rotation matrix R is calculated from the quatern_* parameters.
 This calculation is described below.

 The scaling factor qfac is either 1 or -1.  The rotation matrix R
 defined by the quaternion parameters is "proper" (has determinant 1).
 This may not fit the needs of the data; for example, if the image
 grid is
   i increases from Left-to-Right
   j increases from Anterior-to-Posterior
   k increases from Inferior-to-Superior
 Then (i,j,k) is a left-handed triple.  In this example, if qfac=1,
 the R matrix would have to be

   [  1   0   0 ]
   [  0  -1   0 ]  which is "improper" (determinant = -1).
   [  0   0   1 ]

 If we set qfac=-1, then the R matrix would be

   [  1   0   0 ]
   [  0  -1   0 ]  which is proper.
   [  0   0  -1 ]

 This R matrix is represented by quaternion [a,b,c,d] = [0,1,0,0]
 (which encodes a 180 degree rotation about the x-axis).

 METHOD 3 (used when sform_code > 0):
 -----------------------------------
 The (x,y,z) coordinates are given by a general affine transformation
 of the (i,j,k) indexes:

   x = srow_x[0] * i + srow_x[1] * j + srow_x[2] * k + srow_x[3]
   y = srow_y[0] * i + srow_y[1] * j + srow_y[2] * k + srow_y[3]
   z = srow_z[0] * i + srow_z[1] * j + srow_z[2] * k + srow_z[3]

 The srow_* vectors are in the NIFTI_1 header.  Note that no use is
 made of pixdim[] in this method.

 WHY 3 METHODS?
 --------------
 Method 1 is provided only for backwards compatibility.  The intention
 is that Method 2 (qform_code > 0) represents the nominal voxel locations
 as reported by the scanner, or as rotated to some fiducial orientation and
 location.  Method 3, if present (sform_code > 0), is to be used to give
 the location of the voxels in some standard space.  The sform_code
 indicates which standard space is present.  Both methods 2 and 3 can be
 present, and be useful in different contexts (method 2 for displaying the
 data on its original grid; method 3 for displaying it on a standard grid).

 In this scheme, a dataset would originally be set up so that the
 Method 2 coordinates represent what the scanner reported.  Later,
 a registration to some standard space can be computed and inserted
 in the header.  Image display software can use either transform,
 depending on its purposes and needs.

 In Method 2, the origin of coordinates would generally be whatever
 the scanner origin is; for example, in MRI, (0,0,0) is the center
 of the gradient coil.

 In Method 3, the origin of coordinates would depend on the value
 of sform_code; for example, for the Talairach coordinate system,
 (0,0,0) corresponds to the Anterior Commissure.

 QUATERNION REPRESENTATION OF ROTATION MATRIX (METHOD 2)
 -------------------------------------------------------
 The orientation of the (x,y,z) axes relative to the (i,j,k) axes
 in 3D space is specified using a unit quaternion [a,b,c,d], where
 a*a+b*b+c*c+d*d=1.  The (b,c,d) values are all that is needed, since
 we require that a = sqrt(1.0-(b*b+c*c+d*d)) be nonnegative.  The (b,c,d)
 values are stored in the (quatern_b,quatern_c,quatern_d) fields.

 The quaternion representation is chosen for its compactness in
 representing rotations. The (proper) 3x3 rotation matrix that
 corresponds to [a,b,c,d] is

       [ a*a+b*b-c*c-d*d   2*b*c-2*a*d       2*b*d+2*a*c     ]
   R = [ 2*b*c+2*a*d       a*a+c*c-b*b-d*d   2*c*d-2*a*b     ]
       [ 2*b*d-2*a*c       2*c*d+2*a*b       a*a+d*d-c*c-b*b ]

       [ R11               R12               R13             ]
     = [ R21               R22               R23             ]
       [ R31               R32               R33             ]

 If (p,q,r) is a unit 3-vector, then rotation of angle h about that
 direction is represented by the quaternion

   [a,b,c,d] = [cos(h/2), p*sin(h/2), q*sin(h/2), r*sin(h/2)].

 Requiring a >= 0 is equivalent to requiring -Pi <= h <= Pi.  (Note that
 [-a,-b,-c,-d] represents the same rotation as [a,b,c,d]; there are 2
 quaternions that can be used to represent a given rotation matrix R.)
 To rotate a 3-vector (x,y,z) using quaternions, we compute the
 quaternion product

   [0,x',y',z'] = [a,b,c,d] * [0,x,y,z] * [a,-b,-c,-d]

 which is equivalent to the matrix-vector multiply

   [ x' ]     [ x ]
   [ y' ] = R [ y ]   (equivalence depends on a*a+b*b+c*c+d*d=1)
   [ z' ]     [ z ]

 Multiplication of 2 quaternions is defined by the following:

   [a,b,c,d] = a*1 + b*I + c*J + d*K
   where
     I*I = J*J = K*K = -1 (I,J,K are square roots of -1)
     I*J =  K    J*K =  I    K*I =  J
     J*I = -K    K*J = -I    I*K = -J  (not commutative!)
   For example
     [a,b,0,0] * [0,0,0,1] = [0,0,-b,a]
   since this expands to
     (a+b*I)*(K) = (a*K+b*I*K) = (a*K-b*J).

 The above formula shows how to go from quaternion (b,c,d) to
 rotation matrix and direction cosines.  Conversely, given R,
 we can compute the fields for the NIFTI-1 header by

   a = 0.5  * sqrt(1+R11+R22+R33)    (not stored)
   b = 0.25 * (R32-R23) / a       => quatern_b
   c = 0.25 * (R13-R31) / a       => quatern_c
   d = 0.25 * (R21-R12) / a       => quatern_d

 If a=0 (a 180 degree rotation), alternative formulas are needed.
 See the nifti1_io.c function mat44_to_quatern() for an implementation
 of the various cases in converting R to [a,b,c,d].

 Note that R-transpose (= R-inverse) would lead to the quaternion
 [a,-b,-c,-d].

 The choice to specify the qoffset_x (etc.) values in the final
 coordinate system is partly to make it easy to convert DICOM images to
 this format.  The DICOM attribute "Image Position (Patient)" (0020,0032)
 stores the (Xd,Yd,Zd) coordinates of the center of the first voxel.
 Here, (Xd,Yd,Zd) refer to DICOM coordinates, and Xd=-x, Yd=-y, Zd=z,
 where (x,y,z) refers to the NIFTI coordinate system discussed above.
 (i.e., DICOM +Xd is Left, +Yd is Posterior, +Zd is Superior,
      whereas +x is Right, +y is Anterior  , +z is Superior. )
 Thus, if the (0020,0032) DICOM attribute is extracted into (px,py,pz), then
   qoffset_x = -px   qoffset_y = -py   qoffset_z = pz
 is a reasonable setting when qform_code=NIFTI_XFORM_SCANNER_ANAT.

 That is, DICOM's coordinate system is 180 degrees rotated about the z-axis
 from the neuroscience/NIFTI coordinate system.  To transform between DICOM
 and NIFTI, you just have to negate the x- and y-coordinates.

 The DICOM attribute (0020,0037) "Image Orientation (Patient)" gives the
 orientation of the x- and y-axes of the image data in terms of 2 3-vectors.
 The first vector is a unit vector along the x-axis, and the second is
 along the y-axis.  If the (0020,0037) attribute is extracted into the
 value (xa,xb,xc,ya,yb,yc), then the first two columns of the R matrix
 would be
            [ -xa  -ya ]
            [ -xb  -yb ]
            [  xc   yc ]
 The negations are because DICOM's x- and y-axes are reversed relative
 to NIFTI's.  The third column of the R matrix gives the direction of
 displacement (relative to the subject) along the slice-wise direction.
 This orientation is not encoded in the DICOM standard in a simple way;
 DICOM is mostly concerned with 2D images.  The third column of R will be
 either the cross-product of the first 2 columns or its negative.  It is
 possible to infer the sign of the 3rd column by examining the coordinates
 in DICOM attribute (0020,0032) "Image Position (Patient)" for successive
 slices.  However, this method occasionally fails for reasons that I
 (RW Cox) do not understand.
```

## OME-Zarr

This is what the zarr hierarchy looks like:
```
└── <basename>.ome.zarr
    │
    ├── .zgroup               # Each volume is a Zarr group, of arrays.
    ├── .zattrs               # Group level attributes are stored in the .zattrs
    |                         # file include the OME "multiscales" key.
    |                         # In addition, the group level attributes
    │                         # may also contain "_ARRAY_DIMENSIONS" for
    |                         # compatibility with xarray if this group directly
    |                         # contains multi-scale arrays.
    │
    ├── 0                     # Each multiscale level is stored as a separate
    |                         # Zarr array, which is a folder containing chunk
    │   ...                   # files which compose the array.
    |
    └── n                     # The name of the array is arbitrary with the
        |                     # ordering defined by the "multiscales" metadata,
        │                     # but is often a sequence starting at 0.
        │
        ├── .zarray           # All image arrays must be up to 5-dimensional
        │                     # with the axis of type time before type channel,
        |                     # before spatial axes.
        │
        └─ t                  # Chunks are stored with the nested directory
           └─ c               # layout. All but the last chunk element are stored
              └─ z            # as directories. The terminal chunk is a file.
                 └─ y         # Together the directory and file names provide the
                    └─ x      # "chunk coordinate" (t, c, z, y, x), where the
                              # maximum coordinate will be dimension_size / chunk_size.
```

This is what the multiscales metadata look like:
```yaml
{
    "multiscales": [
        {
            "version": "0.4",
            "name": "example",
            "axes": [
                {"name": "t", "type": "time", "unit": "millisecond"},
                {"name": "c", "type": "channel"},
                {"name": "z", "type": "space", "unit": "micrometer"},
                {"name": "y", "type": "space", "unit": "micrometer"},
                {"name": "x", "type": "space", "unit": "micrometer"}
            ],
            "datasets": [
                {
                    "path": "0",
                    "coordinateTransformations": [{
                        // the voxel size for the first scale level (0.5 micrometer)
                        "type": "scale",
                        "scale": [1.0, 1.0, 0.5, 0.5, 0.5]
                    }]
                },
                {
                    "path": "1",
                    "coordinateTransformations": [{
                        // the voxel size for the second scale level (downscaled by a factor of 2 -> 1 micrometer)
                        "type": "scale",
                        "scale": [1.0, 1.0, 1.0, 1.0, 1.0]
                    }]
                },
                {
                    "path": "2",
                    "coordinateTransformations": [{
                        // the voxel size for the third scale level (downscaled by a factor of 4 -> 2 micrometer)
                        "type": "scale",
                        "scale": [1.0, 1.0, 2.0, 2.0, 2.0]
                    }]
                }
            ],
            "coordinateTransformations": [{
                // the time unit (0.1 milliseconds), which is the same for each scale level
                "type": "scale",
                "scale": [0.1, 1.0, 1.0, 1.0, 1.0]
            }],
            "type": "gaussian",
            "metadata": {
                "description": "the fields in metadata depend on the downscaling implementation. Here, the parameters passed to the skimage function are given",
                "method": "skimage.transform.pyramid_gaussian",
                "version": "0.16.1",
                "args": "[true]",
                "kwargs": {"multichannel": true}
            }
        }
    ]
}
```

## NIfTI-Zarr

We just add a zarr-array in the group, named nifti. Its datatype is "u1" (i.e., raw bytes) and it contains the raw header.
```
└── <basename>.nii.zarr       # A NIfTI volume converted to Zarr.
    │
    ├── .zgroup
    ├── .zattrs
    |
    ├── nifti                 # The NIfTI header is stored as a raw array
    │   ├── .zarray           # of bytes, with an optional JSON form stored in
    │   ├── .zattrs           # the associated .zattrs file.
    │   └── 0
    │
    ├── 0
    │   ...
    └── n
```

To ensure that the binary chunk can be parsed by existing nifti readers, we have 
additional rules about its encoding:
```
<basename>.nii.zarr/nifti/.zarray
{
    "zarr_format": 2,       # MUST be 2
    "dtype": "u1",          # MUST be "u1" or "S{length:d}"
    "shape": [348],         # MUST be header length if `dtype="u1"` else 1
    "chunks": [348],        # MUST be same as `"shape"`
    "compressor": {
        "id": "zlib",       # MUST be `{"id": "zlib", "level": 0..9}` or `null`
        "level": 9          # MUST be in 0..9, if "zlib"
    },
    "order": "C",           # UNUSED ("F" or "C")
    "fill_value": null,     # UNUSED
}
```
