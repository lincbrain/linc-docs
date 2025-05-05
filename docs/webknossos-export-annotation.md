# Export Annotations from Webknossos

The following steps provide instructions for downloading annotations from webknossos. 

1. Go to Annotations tab and click open on target annotation


    ![](img/webknossos_annotation.png)

2. Click on Download on dropdown menu

    ![](img/download.png)

3. Select "Include volume annotations as WKW" on pop out box and click on Download

    ![](img/download2.png)

4. Unzip downloaded annotation file 

5. Unzip data_Volume.zip under annotation folder 

    ![](img/data_volume.png)

6. Run the [webknossos_annotation.py](https://github.com/lincbrain/linc-convert/blob/main/linc_convert/modalities/wk/webknossos_annotation.py) script to save into the OME-Zarr format following the `czyx` direction. 

   
