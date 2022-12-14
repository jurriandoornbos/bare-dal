FROM jupyter/scipy-notebook:python-3.9
LABEL drone-ml dev
ARG JUPYTERHUB_VERSION=3.0.0
RUN pip3 install --no-cache \
    jupyterhub==$JUPYTERHUB_VERSION

# Rastervision Copy-paste
# We need to install GDAL first to install Rasterio on non-AMD64 architectures.
# The Rasterio wheels contain GDAL in them, but they are only built for AMD64 now.
RUN conda config --append channels conda-forge


RUN pip3 install pyodm

RUN conda install -y -c conda-forge rioxarray==0.13.1 geopandas==0.12.1

RUN git pull https://github.com/ICAERUS-EU/ddal.git

#RUN pip3 install rastervision==0.20

CMD ["jupyterhub-singleuser"]