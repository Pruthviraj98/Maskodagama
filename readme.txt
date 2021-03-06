Process Explained:

	load image batches
	
	convert the images to grayscale
	
	extract hog features from those images
	
	train the features on nn

Hog (Histogram Oriented Gradient) feature extraction features:
ref:learnopencv.com/histogram-of-oriented-gradients/
	
	HOG is a feature descriptor. converts images of size [width, height, 3(channels)] into a feature vector or a array of length n.

	In case of HOG, distribution of directions of gradients (x, y derivatives) are used as features. This is useful because the magnitude of gradients in greater in edges and corners. Because that's where the intensity of light changes abruptly and edges pack lots of info about the object.

	Hog features are calculated on a patch of image (of any size). Conventionally, multiple such patches (of different size) at multiple locations of the same image are considered for extracting the feature. 

	So, steps are: 
	1. (Preprocessing)- selecting the patches from the image. 
	2. Calculating the gradient Images
		calculate dx (i.e. gx), dy (i.e. gy), magnitude (i.e. sqrt(gx**2+gy**2)),
		direction (i.e. argtan(gy/gx))

		This gx and gy can be calculated by convoluting the patch with the filters like gaussian mask/prewitt  mask.
	note: x-grad= fires on vertical lines
		  y-grad= fires on horizontal lines
		  magnitude= where there is sharp change in intensity
		  For color images, the gradients of the three channels are evaluated ang the max of the gradient pixel among three at every pixel is chosen.
	3. Calculate Histogram of Gradients in 8*8 cells
		First the image is converted to 8*8 cells. Further the histogram of gradients is calculated for each of those cells

		why 8*8? = 8*8 size helps us gaining more interesting features (face, top of head. etc) and also to gain a compact representation from HOG feature descriptor. In an 8*8 patch, there are 8*8*3 pixels (192). The gradient of this pack contains 2 values. i.e. magnitude and direction. So, per pixel in a channel, no. of values = 8*8*2 =128 values. These 128 numbers are represented in -bin histogram that inturn can be stored as an array of 9 numbers. This also makes model more robust to noise. The direction of arrows points to the direction of change in intensity and the magnitude shows how big the difference is.

	note: unsigned gradients: angle is in range 0-180 (negatives (>180) are clipped to 180)
	 	
	 	So, here, we create histogram of gradients in 8*8 cells i.e. creating the 9 bbins corresponding to angles 0, 20, 40, ....160.

	 	note: look into the ref link for illustration how the both grad direction (angle) pixel values are compared with gradeint magnitude values and are split into bins. A bin is selected based on the direction, and the vote ( the value that goes into the bin ) is selected based on the magnitude.   

	 4. 16*16 Block Normalization
	 	The gradients of the images are sensiive to the change in light intensity. Ex: if we change the pixel value by half, the gradient too decreases by half inturn decreasing the histogram value by half. Thus we need the normalization process to eradicate this sensitivity.
	 	For example: 
	 	Considering the RGB value of  a pixel: [128, 64, 32], length= sqrt((128)**2 + (64)**2 + (32)**2) is 146.32. dividing th vector by this value, gives [0.87, 0.43, 0.22]. Considering another vector, twice of first vector [256, 128, 64], by finding the normalized vector of it, we get the same [0.97, 0.43, 0.22] unchanged.  
	 	For one block we have 9*1 histogram, so for 4 blocks, i.e. for 16*16 block, we have 9*4=36 histogram values. Normalizing that, we have normalized 36*1 vector. Further, we move the window further (refer the link :https://www.learnopencv.com/wp-content/uploads/2016/12/hog-16x16-block-normalization.gif). 


	5. Calculating the HOG feature vector

		Here, all the 36*1 vectors are concatenated to form a bigger vector.

		Ex: How many different positions did the 16*16 window moved in the picture?
		(from the link there were 7hor*15vert).

		so totally 7*15=105 positions.

		Hence, 36*1*105=3780 valued vector conatenated to be returned as a hog vector.


