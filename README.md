# Galloway faceswap 

## Problem Description

> Our day-to-day work requires a lot of hands-on experiments. The purpose of this task is to test
your ability with learning new tech and applying it toward a real use case. This is not a
coding-heavy task. This task mostly involves config and setup work and some experimentation
to tune parameters.

> Please use https://github.com/deepfakes/faceswap to swap the person in this video,
http://codefor.cash/img/bermi_video.mp4, into any Latina celebrity. The higher the quality, the better..
Your result should be the modified video and a GitHub repo with your accompanying code. Think
about how to scale your training data, maybe with Amazon Mechanical Turk templates?

## Process

My code is almost entirely in the jupyter notebook, but there are some additional steps taken worth documenting.

1. The original .mp4 caused problems when I originally tried to render it and it seemed to have errors due to windows treating it weird.

Fixed With: `ffmpeg -i bermi_video.mp4 bermi_video2.mp4`
   
2. I picked Alexandria Ocasio-Cortez due to skin tone and the availability of video with just her face without too much makeup and basic lighting.  Specifically, I used a video of her [inaugural address](https://youtu.be/u7NlJt34XSM), along with the first 100 results of her google image search.

3. Used blender video editor to make a negative mask around her face with keyframes to handle her movement during the video.  It's important to sanitize the video in this manner because deleting false positive faces in the extract is hugely time consuming. I pulled about 3k images from this video (one every 10 seconds) again, just using blender render-as-images.  (this step was a necessarybut hugely time consuming, could be sped upconsiderably with better hardware)

4. I've added another 100 images off google image search using [google-image-search](https://github.com/hardikvasa/google-images-download) python app.  Adding more is a todo (TODO: Install chromium and get 500 instead).

5. Face extraction was done on google colab but would be better handled on a local machine since network transfer slows everything down.  The code is in the notebook under **Extract faces from source** but in detail:

* Upload files.

* Extract faces with `!python3 faceswap/faceswap.py extract -i Drive/bermi_video2.mp4 -o Drive/bermi_extract -D s3fd`

* Download extracted faces and delete all the false positives.

* Re-upload the fixed faces.

* Remove eliminated faces from alignments.json with `!python3 faceswap/tools.py alignments -j remove-faces -a Drive/bermi_video2_alignments.json -fc Drive/bermi_extract_fixed`

* Remove any frames missing alignments with `!python faceswap/tools.py alignments -j leftover-faces -a Drive/bermi_video2_alignments.json -fc Drive/bermi_extract_fixed -o move`

6. The original notebook used the `-t villain` model for training but I wasn't happy with the results.  I'm now using `-t realface` with the `-wl` option and the results are better but it requries learning the alignment.json dance above.  I'm not sure how long I trained the current model, but it was at least 12 hours.

7. The video would be improved a lot by not converting after the first 4 seconds but left it in since it's not 'production'.

## ROOM FOR IMPROVEMENT

1. Try Ariana Grande.  There's a [massive face corpus](https://mrdeepfakes.com/forums/thread-ariana-grande-faceset-5-1-big-update-3-17-19) of her (warning: porno site).

2. Try using [DFL](https://colab.research.google.com/github/chervonij/DFL-Colab/blob/master/DFL_Colab_Demo.ipynb) and the SAE model (doesn't seem available in deepfakes).

