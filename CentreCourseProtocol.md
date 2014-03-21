# Analysis of High-Throughput Microscopy Image Data

BioIT-EMBL Centres Joint Training Week, Mar. 26, 2014

Kota Miura

Centre for Molecular and Cellular Imaging (CMCI), EMBL

<miura@embl.de>

TA: Christoph Schiklenk

Haering Lab, Cell Biology & Biophysics, EMBL



##Course Outline

1. Aim: Learn how to process and analyze high content image data by Fiji scripting
2. Introduction: Background of data
3. Processing and analyzing using Fiji GUI
   - We first try to segment nucleus image and do measurement using GUI, to know how the algorithm works.    
4. Python Crash Course
   - We want to write a script that does processing and measurements: for this we first need to learn  basics of python syntax.   
5. Fiji scripting in Jython
   - Jython is a Java-implemented Python, allows to use full functionality of Fiji by scripting. We learn how to write Jython script to do our task automatically.  
6. Submitting Jobs (executing Jython script) to the cluster.
   - As there are many images, we use cluster to parallelize the computation, to get results in a short time. 

## Introduction

Image based screening (high content screening) is one of the modern methods for screening genes involved in a biological process. The main idea is to sytematically acquire a huge amount of data with various treatments, and try to isolate treatments that affects the target biological process. 

Dataset we use in this course is from a scretory pathway screening. Proteins synthesized in endoplasmic reticulum are transported first to the Golgi, where proteins are added with modifications, and then to the plasma membrane or other desitned intracellular structures. 

VSVG is a model protein used for studying this transport system and is a heat-sensitive protein.  This protein could beaccumulated in / released from ER by temperature shifting, to experimentally analyze the process of intracellular transport from ER to the plasma membrane. 

To study the transport process, it is desirable to take time lapse sequence to follow the changes in VSVG localization within cell. However, it is sufficient to assess the transport process by taking a snapshot at a defined time point after releasing the protein from ER and check where the protein is located. In control condition, proteins should be localized in the plasma membrane after successful transports. If the transport out from the ER is interferred due to suppression of the component of transport, proteins should be stably located in the ER. Likewise, finding proteins in the Golgi suggests that transport machinary from Golgi to the plasma membrane is blocked. 

With this snapshot strategy, it is possible to systematically prepare a set of siRNAs and drug treatments to test their effect on vesicle transport system: the high content image screening. 

In this course, we will try to analyze all images from different types of treatments automatically, evaluate and compare the difference in the effeciency of transport. This COULD be done manually, but we have huge number of various treatments: approximately  of images to analyze. To not to end the rest of our life clicking images, we will completely automate the image processing and  analysis to extract transport parametes and output results as a huge list of parameter table. 

This table then is studied using statistical techniques to distinguish treatments that are largely affecting the transport system. 

Followings are the literatures published with this dataset. 

- Liebel, U., Starkuviene, V., Erfle, H., Simpson, J.C., Poustka, A., Wiemann, S., Pepperkok, R., 2003. A microscope-based screening platform for large-scale functional protein analysis in intact cells. FEBS Lett. 554, 394–8. <http://www.ncbi.nlm.nih.gov/pubmed/17275941>

- Simpson, J.C., Cetin, C., Erfle, H., Joggerst, B., Liebel, U., Ellenberg, J., Pepperkok, R., 2007. An RNAi screening platform to identify secretion machinery in mammalian cells. J. Biotechnol. 129, 352–65. <http://www.ncbi.nlm.nih.gov/pubmed/17275941>

- Simpson, J.C., Joggerst, B., Laketa, V., Verissimo, F., Cetin, C., Erfle, H., Bexiga, M.G., Singan, V.R., Hériché, J.-K., Neumann, B., Mateos, A., Blake, J., Bechtel, S., Benes, V., Wiemann, S., 
Ellenberg, J., Pepperkok, R., 2012. Genome-wide RNAi screening identifies human proteins with a regulatory function in the early secretory pathway. Nat. Cell Biol. 14, 764–74. <http://www.ncbi.nlm.nih.gov/pubmed/22660414>

## Methods (Image Analysis)

As we are interested in the attenuation or the enhancement of the protein transport machinary from ER to the plasma membrane via the Golgi apparatus, we compare the protein density in ER, the Golgi and the plasma membrane. For this, we use three different images. 

The minimal detaset consists of three images:

1. Dapi signal: nucleus image
2. cy3 signal: VSVG signal
3. Immunostaining signal: VSVG signal exposed to the extracellular space. 

Dapi signal is used for locating individual cells and marking them. VSVG signal tells us where the VSVG protein is located within the cell. When they are at the Golgi, they are concentrated close to the nucleus. When they are in ER and the plasma membrane, they are evenly distributed within the cell. To check the VSVG protein specifically localized to the plasma membrane, the third image from immunostaining of the VSVG protein is used: the signal only appears when the protein is exposed to the extracellular space.

###Prescreening

There are bad images just because of simple failures during acquisition, such as

- out of focus
- over exposure

We exclude these images by prescreening images. Standard deviation of pixel intensity in each image is measured, and those with extreme values will be excluded from the analysis. For this, a black list of file names which will be used in the main analysis is created.

We probably will not explain this part in detail, just show you how it was done.  

###Nucleus Segmentation

To measure these values, we first segment each nucleus using atutomatic local intensity thresholding (Bernsen algorithm). Then the rim of the nucleus is dilated outwards to define an approximate area to define the region of **Cell Area**. Within this region, we call the dilated region (Cell Area - Nucelus Area) as the **Juxta-Nuclear** cytoplasmic region. 

(Figure required here)

By measuring the intensity ratio of those at the plasma membrane and those at the Golgi (the transport ratio **T**), we assess the effeciency of transport.

### Transport Ratio
The transport ratio (T) calculated in Simpson (2007) is as follows:

Mean intensity of PM signal / Mean Intensity of VSVG signal. 

If **T** is smaller than control cells, transport is blocked. If it is larger than control, then the transport is accelerated. 

## Resources

###Fiji

Fiji is located at

	/g/almf/software/bin2/fiji

Please put the following lines in your .bashrc or .bash_profile

	export PATH=/g/almf/software/bin2:$PATH
	export PATH=/g/almf/software/bin/java/jdk1.6.0_45/bin:$PATH

Then 

	source .bashrc

or

	source .bash_profile
	
###Data

All data are located under

	/g/data/bio-it_centres_course/data/VSVG/

Plates are numbered such that

	plateID-Number--year-month-date

For exmaple:

	0001-03--2005-08-01

This means that this is the plate ID 1, third experiment (same plate configuration is repeated at least three times each). We analyze plateID from 1 to 159. 

Within each of these folders, images are stored under 
	
	data/

###Scripts

The script for processing images for a single plage is

	/g/almf/software/scripts2/measTransportBatch3.py
	
Prescreening script is 

	/g/almf/software/scripts2/listFocusedImagesV2.py
	
A script for generating job array is
	
	
## Workflow: GUI

The protocol below is described in Simpson (2007).

###Background subtraction 

Background subtraction will only be done with PM and VSVG images. In the final script, the funciton name is **backgroundSubtraction**

Get minimum and mean intensity of the image.

	[Analyze > Set measurements…] : min and max, mean
	[Analyze> Measure]
		
Results will be shown in the Results Table. We use min and mean values. We call them min1, mean1. 

Then calculate mean of the pixels those with values greater than min1 and less than the mean1. 

	[Analyze > Set measurements…]
		:select min and max, mean, limit to threshold
	[Image > Adjust > threshold…]
		click "Set" and put 
			Lower Threshold Level: min1
			Upper Threshold level: mean1
	[Analyze> Measure]
		results: mean2
	Click "Reset" in Threshold GUI.      

Subtract mean2 value from the image. 
     
	[Process > Math > Subtract]: use "mean2". 

###nucleus segmentation

Nucleus segmentration. We only do this with Dapi image. Corresponding function in the script is **nucleusSegmentation(imp2)**.
 
	[Image > Type > 8bit]
	[Process > Filters > Gaussian Blur...]
		: radius=2.0
	[Image . Adjust > Auto Local Threshold]
		: algorithm, Bernsen, other parameters stay same
   
Pre-filtering by size and circularity

	[Analyze > Set Measurements]
          :select  area, standard deviation, shape descriptors, integrated density
	[Analyze > Analyze Particles…]
		:set area 20 - 1000
		:set circularity 0.8 - 1.0
		:show: Masks
		:check Display Results, Clear results, Exclude on Edges, Include Holes
Rename the resulted image as impNuc

	[Image > Rename]

Duplicate the resulted image and then invert LUT

	[Image > Duplicate…]
	[Image > Lookup Tables > Invert LUT]
          
Dilate segmented nucleus by 15 pixels (to check if nucleus will merge)
	
	[Process > Binary > Options] set iterations to 15, Check "Black Background"
	[Process > Binary > Dilate]

(called imp2)


Particle analysis 1 (with imp2) 

	[Analyze > Analyze Particles…]
		:set area 20 - 10000
		:set circularity 0.8 - 1.0
		:show: None
		:check Display Results, Clear results, Exclude on Edges, Include Holes

q1, q3, offset calculation: Copy and paste <https://gist.github.com/cmci/9641262>

     Example results: 2926.0 3854.0 1392.0
     min2 = q1 - offset (= 2462)
     max2 = q3 + offset (= 5246)


Particle analysis 2 (remove outlier)

	with imp2, 
	[Analyze > Analyze Particles…]
		:set area min2 - max2
		:set circularity 0.8 - 1.0
		:show: Mask
		:check Display Results, Clear results, Exclude on Edges, Include Holes
     
	output: the mask. rename it as impDilatedNuc

Filter original nulcues

     [Process > Image Calculator …]
          AND operation for impNuc and impDilatedNuc


###Check Segmentation Results. 

     Visual inspection. We terminate analysis if there is no nucleus segmented. 


###Generate nuclus ROIs from segmented nucleus. Store them as a list. 

     [Image > Selection > Create Selection]
     [Image > Selection > Make Inverse]
     [Image > Selection > To ROImanager]

###Roi modifications

     In ROI manager
     a. Generate All Cell ROIs (enlarge)
         Activate the ROI above, then 
         [Image > Selection > Enlarge…] : 15 pixels
          click "Add". Second ROI appears in the ROI manager list. 
          this will be the "All cell"
     b. Generate JuxtaNuclear ROIs (enlarged minus nucleus)
          select both ROIs, and then [More >> XOR]. 
          click "Add". Third ROI appears in the listing. 
          This ail be the zone surrounding nucleus. 


### Measurements
Do measurements using modified ROIs (measureROIs).

     select corresponding ROI and 
     
     [Analyze > Measure]
     
     a. enlarged: all cell plasma membrane
     b. ring rois: juxtanuclear signal for VSVG
     c. all cell VSVG intensity


### Transport Effeciency

Calculate transport efficiency and add the results to results table. All results from single plate are merged and placed in a file, but we do not do this manually. 

## Workflow: Python Primer

We do this interactively. Below is a list of Python funtions we conver.

````
print
print with comma separated

for
if, else, break

list
     initialization
     range
     sorted(list)
     slicing
     l.append()
     len

     allcellA = [roiEnlarger(r) for r in nucroiA]

     enumerate
     extend()


String
     concatenation
     f.endswith()
     
dictionary
     initialization
     d['a'] = 3

Accessing files
     path / file managements
          os.path.listdir
          os.path.isfile
          os.path.join

Python regex
          re.compile
          re.search
          re.group
          
I/O 
f = open(path, 'rb')
sys.argv

csv.reader


functions
     return
     multiple returns
````

## Workflow: Fiji Jython Scripting

### Prescreening of Images.

<https://gist.github.com/cmci/9684453>

Prescreen results are saved under

	/g/data/bio-it_centres_course/data/VSVG/prescreen

### Nucleus segmentation

A script for processing single image dataset is:

<https://gist.github.com/cmci/9644827>

This script segments nucleus. 

### ROI generator

### Measurements

#### Transport Ratio



#### Haralick Features

## Final Code
