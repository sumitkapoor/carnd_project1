#**Finding Lane Lines on the Road** 

##Writeup Template

###You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

###1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps. First, I converted the images to grayscale, then I .... 

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

[[ 0.5       ]
 [ 0.5625    ]
 [-0.07692308]
 [-0.86666667]
 [-0.80952381]
 [-0.68085106]
 [-0.72727273]
 [-0.75      ]
 [ 0.11111111]
 [-0.8       ]
 [-0.74358974]
 [-0.64      ]
 [-0.83333333]
 [-0.70588235]
 [-0.82352941]
 [-0.75      ]
 [ 0.53846154]
 [ 0.61538462]
 [ 0.6       ]
 [-0.78571429]]


[[-0.79166667]
 [-0.72571429]
 [ 0.5       ]
 [-0.72307692]
 [-0.71794872]
 [-0.72727273]
 [-0.66037736]
 [-0.63636364]
 [-0.73076923]
 [ 0.61111111]
 [ 0.8       ]
 [ 0.62857143]
 [-0.65625   ]
 [-0.72727273]
 [-0.8       ]
 [-0.6       ]
 [-0.75      ]]
 
 [[-0.64912281]
 [-0.62285714]
 [-0.63636364]
 [-0.6875    ]
 [-0.625     ]
 [-0.7       ]
 [-0.64285714]
 [-0.65384615]
 [ 0.7       ]
 [ 0.15384615]
 [-0.66666667]
 [ 0.8       ]
 [-0.84615385]
 [ 0.61904762]
 [-0.6875    ]
 [-0.8       ]
 [-0.66666667]
 [ 0.57894737]
 [-0.72727273]]
 left_m = -0.629131367585  right_m = -0.321840153633
left_c = 615.184264338  right_c = 578.417799966
left_y = 333.962543027  right_y = 379.198744867  min y = 333.962543027
left_y = 540.946762963  right_y = 519.199211697  max y = 540.946762963
If you'd like to include images to show how the pipeline works, here is how to include an image: 

![alt text][image1]


###2. Identify potential shortcomings with your current pipeline


One potential shortcoming would be what would happen when ... 

Another shortcoming could be ...


###3. Suggest possible improvements to your pipeline

A possible improvement would be to ...

Another potential improvement could be to ...