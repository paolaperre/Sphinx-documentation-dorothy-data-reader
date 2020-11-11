.. Drothy data reader docs documentation master file, created by
   sphinx-quickstart on Tue Nov 10 20:32:54 2020.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to Drothy data reader docs's documentation!
===================================================
Requirements
====================================================
To start using the library, you must download
the clinical readings and the images that you want to
analyse, and save them in your computer(you will
need the path of them to run the code).
After that, you need to install the repository::

   pip install git+https://github.com/tb-brics/dorothy-data-reader.git

Doing that, you will have all the needs to start using
the library

Running the library
====================================================
First of all, if you want to get information about the images and
clinical readings, you need to import the 'Dataset' corresponding
to the country you want the information from(China, Montgomery, India).::

   from xrayreader import Dataset

To get information from a specific dataset, you will need to use the path where the images are stored.
This will show the name of the dataset which you are getting information from, also
the name of the images and of the clinical readings.::

   data = Dataset(name = 'name of the country you want', path = 'path where the information is stored')
   data.get_data()

Extending the Library
====================================================
To add new databases in the library, the first step is:
extend a file within the metadatareader module.
This file must have a  ``class Reader`` , which will inherit the functions and methods of
``ReaderBase`` (get_filenames and parse_files - the last one is responsible to store
the patient data -age, gender and report) and will be able to get the data from the clinical
readings and return them into a list.::

   class ReaderBase:
    """
    This class stores the filenames into a list.
    """
    suffix = "*"

    def __init__(self, **kwargs):
        path = kwargs.get("path")
        if path:
            self.folder = os.path.join(path, self.suffix)
        self.filenames = []

    def get_filenames(self):
        """
        Return the name of the file in which the report is stored.
        """
        if self.folder:
            self.filenames = glob.glob(self.folder)
        return self.filenames

    def parse_files(self):
        """
        This function stores patient data(age, gender, report) in a list 'data_china'.
        """

Inside of the ``Reader``, functions must be established to handle the specifics of the
data you want to insert.


The next step is to extend a file, now inside of images module.
This file must have a `` class Reader``, that inherits the methods and functions
set out in ``ReaderBase`` (like 'get_filenames' and 'get_data').
Using this, it will be able to get the filenames of the images and return them.::

   class ReaderBase:
    """
    This class stores the filenames into a list.
    """
    suffix = "*"

    def __init__(self, **kwargs):
        self.folder = None
        path = kwargs.get("path")
        if path:
            self.folder = os.path.join(path, self.suffix)

    def get_filenames(self):
        """
        Return the name of the file in which the report is stored.
        """
        if self.folder:
            filenames = glob.glob(self.folder)
        return filenames

    def get_images(self):
        """
        Returns the name of the files where the images are stored.
        """
        return [Image(filename=f) for f in self.get_filenames()]


   class Reader(ReaderBase):
    """
    Get images from image data set
    """

After that, it is necessary to create a class inside of data.py, which
will inherit the functions and methods of ``DatasetBase`` and through ``DatasetBaseInterface``
recognize the path used and return the desired data and information from
clinical readings and images.::

   class DatasetBase(DatasetBaseInterface): #pylint: disable=too-few-public-methods
    """
     stores the path where the files are saved
      so that the data can be read.
    """
    dataset = None
    metadata_folder = None
    images_folder = None

    def __init__(self, **kwargs):
        self.path = kwargs.get("path")
        self.metadata_path = os.path.join(self.path,
                                          self.metadata_folder)
        self.images_path = os.path.join(self.path,
                                        self.images_folder)

    def get_data(self):
        metadata_reader = importlib.import_module(
                f"xrayreader.metadatareader.{self.dataset}").Reader
        images_reader = importlib.import_module(
                f"xrayreader.images.{self.dataset}").Reader

        data = {"data": {}}
        data['data']['dataset'] = self.dataset
        if self.images_folder:
            data['data']['images'] = images_reader(path=self.images_path).get_images()
        if self.metadata_folder:
            data['data']['metadata'] = metadata_reader(path=self.metadata_path).parse_files()
        return data

If the data does not have reports and the information about whether or not the patient
has tb is in the filename of the image, this should be resolved in this dataset.



.. toctree::
   :maxdepth: 2

   modules
   xrayreader
   xrayreader.images
   xrayreader.metadatareader
   README.md


   :caption: Contents:
   modules

   Readme <README.md>



Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
