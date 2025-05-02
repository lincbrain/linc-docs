# LINC Dashboard

The [LINC Dashboard](https://dashboard.lincbrain.org) summarizes all data on [lincbrain.org](https://lincbrain.org).

## Developer documentation

### Architecture

- The front-end is built with the Streamlit framework and deployed with Streamlit's Community Cloud.
- The metadata is extracted using a GitHub Action and the CSV files are uploaded to a private AWS S3 bucket.

### Run application locally

Follow the instructions below to run the application locally:
1. Fork and clone the [linc-dashboard](https://github.com/lincbrain/linc-dashboard) repository.
2. Install the `requirements.txt`.
3. Run the application with `streamlit run app.py`.
4. Run the script to extract metadata with `python extract_metadata.py`.  Note, by default the application displays the data from S3.
