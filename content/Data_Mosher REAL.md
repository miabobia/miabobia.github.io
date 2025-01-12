#data_mosher #image_processing #video_filters #glitch_art #python #open_cv #PIL 

# Introduction

- I had seen data moshing videos online and was interested in them. Through social media i found this technical paper on data moshing algorithms
- link to datamoshing article
- created software that takes in a series of video and produces a single video with a datamoshing glitch art effect
- technologies:
	- opencv
	- PIL
	- python
	- argparse
	- pathlib
	- os

# Problem Statement

- I found this paper incredibly interesting, but I found it was too advanced for my current skillset with video and image processing. 
- Glitch art and data moshing in particular I think is a very beautiful and underrated art form. I still wanted to create a 'data moshing' effect
- Friend suggested that instead of preprocessing and analyzing video frames like the paper suggest I try post processing the footage.
- From there I attempted to fake the data moshing effect


# Solution Overview
1. we process all the user's system arguments
2. extract all the frames from passed in video files into jpeg images
3. perform datamoshing effects
4. stitch all data moshed frames back into a video and save it to file
5. delete all saved frames from repository to save space (can be enabled with sys arg)

# Technical Deep Dive

How does the datamoshing work?
1. every n frame save a frame
2. compare every subsequent frames to that frame. Depending on how similar each pixel in the new frame is to the old frame. Overwrite the new pixel in the new frame if it is similar enough to the saved frame
3. repeat till all frames are processed

functions for datamoshing and comparing two frames together
```python
def compare_color(color_a: tuple, color_b: tuple, threshold: int = 215) -> bool:
	# compares two colors
	# if the comparison is less than the threshold returns true
	return abs(sum(color_a) - sum(color_b)) <= threshold

def mosh(saved_frame: Image, new_frame: Image) -> Image:
	pixels = new_frame.load()
	saved_pixels = saved_frame.load()
	for i in range(new_frame.size[0]):
		for j in range(new_frame.size[1]):
			if compare_color(pixels[i, j], saved_pixels[i, j]):
				pixels[i, j] = saved_pixels[i, j]	
	return new_frame
```

# What I learned
- how to use the argparse library and handle many system arguments
- how to use opencv
- techniques for image comparison and their limitations
- how to use poetry
- how to handle pathing to files across different operating systems using `PathLib`
```python
def resolve_path(path_str: str) -> bool:
	path = Path(path_str).resolve()
	return path
```

# Future Plans
1. save audio from input videos and stitch it back together for final video
	1. I would have done this already if i didn't need to consider the user being able to set the FPS manually
	2. I would have considered this more for inital release if i didn't need to implement another library FFMPEG
2. implement more system arguments for more fine tuning of data moshing effect. 
	1. instead of a mosh-rate implement a mosh-range so we can add some randomness to the datamoshing so it doesn't save frames to mosh at a constant rate. I would also like to add the sys arg for adding a random seed so videos can be recreated
3. saving meta_data to the output video. I would like a string in the meta_data to show exactly what arguments were used to create the video. The challenge with this idea would be making sure no user information is included if videos are shared. 
4. implement youtube-dl library so youtube links to videos can be provided instead of local video paths
5. implement pytest. I mainly want to use this to test out all the system arguments work consistently. Also to test out that all the cleanup on exit works properly
![[videos/screencast_test.mp4]]
![[https://miabobia.github.io/videos/screencast_test.mp4]]