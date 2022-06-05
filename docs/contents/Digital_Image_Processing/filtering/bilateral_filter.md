## bilateral filter

### 文章
@inproceedings{2002Bilateral,
  title={Bilateral filtering for gray and color images},
  author={ Tomasi, C.  and  Manduchi, R. },
  booktitle={International Conference on Computer Vision},
  year={2002},
}
- Fast O(1) bilateral ﬁltering using trigonometric range kernels
- 

### 核心公式
$$I(x,y)'=G_s(I(x,y))*G_i(I(x,y))$$

 $$pixel_{sum} = \sum_{p\in block}I(x,y)*weight(x,y)$$  
$$weight_{sum} = \sum_{p\in block}G_i(I(x,y)-I(x',y')) * G_s(\sqrt{(x'-x)^2+(y'-y)^2})$$  


### 实现
````
import cv2
import numpy as np
from tqdm import tqdm

""" naive implement """
def gaussian(x, sigma):
    return (1.0/ \
           (2 * np.pi * (sigma**2))) \
            * np.exp(-(x**2)/ \
                    2 * (sigma**2))

def distance(x1, y1, x2, y2):
    return np.sqrt(np.abs((x1-x2)**2 \
                         -(y1-y2)**2) )

def bilateral_filter(image, diameter, sigma_i, sigma_s):
    new_image = np.zeros(image.shape)
    for row in tqdm(range(len(image))):
        print('row ', row)
        for col in range(len(image[0])):
            wp_total = 0
            filtered_image = 0
            for k in range(diameter):
                for l in range(diameter):
                    n_x = row - (diameter/2 - k)
                    n_y = col - (diameter/2 - l)
                    if n_x >= len(image):
                        n_x -= len(image)
                    if n_y >= len(image[0]):
                        n_y -= len(image[0])
                    gi = gaussian(image[int(n_x)][int(n_y)] - image[row][col], sigma_i)
                    gs = gaussian(distance(n_x, n_y, row, col), sigma_s)
                    wp = gi * gs
                    filtered_image = (filtered_image) + (image[int(n_x)][int(n_y)] * wp)
                    wp_total = wp_total + wp
            filtered_image = filtered_image // wp_total
            new_image[row, col] = int(np.round(filtered_image))
    return new_image

````

```
""" speed up """
""" 1 window gaissian LUT """
""" 2 intensity gaussian LUT """
""" 3 window LUT """
diameter, sigma_s, sigma_i = 7, 0.1, 0.1
gi_lut = gaussian(np.arange(0-255, 0+255, 1), sigma = sigma_i)
gs_lut = gaussian(np.arange(0, 0+diameter, 1), sigma = sigma_s)
#window_lut = np.zeros((image.shape[1], image.shape[0], diameter**2))
window_index = diameter >> 1  
map = np.meshgrid(np.linspace(-window_index, window_index, window_index * 2 + 1), \
                     np.linspace(-window_index, window_index, window_index * 2 + 1))
map[0], map[1] = map[0].astype(np.int), map[1].astype(np.int)
gs_weights2d = gaussian(distance(map[0].flatten(), map[1].flatten(), 0, 0), sigma_s)

def bilateral_filter_lut(image, diameter, gs_lut, gi_lut):
    new_image = np.zeros(image.shape)
    for row in tqdm(range( window_index, image.shape[0] - window_index, 1)):
        for col in range( window_index, image.shape[1] - window_index, 1):
            values = image[ row + map[0].flatten(), col + map[1].flatten()]
            gi = gi_lut[values - image[row, col] + 255 - 1]
            wp = gi * gs_weights2d
            pixel = np.sum((values * wp)) / (np.sum(wp))
            #if abs(pixel-image[row, col]) > 10:
            #    print('pixel old ', image[row, col], 'pixel new ', pixel)
            new_image[row, col] = int(np.round(pixel))
    return new_image
```


```
""" joint bilateral filter """


""" bilateral upsampling """


""" todo histogram optimization """



image = cv2.imread("in_img.jpg",0)


#filtered_image_OpenCV = cv2.bilateralFilter(image, 7, 20.0, 20.0)
#cv2.imwrite("filtered_image_OpenCV.png", filtered_image_OpenCV)
#image_own = bilateral_filter_lut(image, 7, 20.0, 20.0)
image_own = bilateral_filter_lut(image, diameter, gs_lut, gi_lut)
cv2.imwrite("filtered_image_own.png", image_own)

cv2.imwrite("diff.png", abs(image - image_own)*5)

"""
The bilateral filter is controlled by important parameters. Two of them are sigma values.
Generally, the bilateral filter gives us more control over image.
If we increment both sigma values at the same time, the bigger sigma values gives us a
more blurred image. If we give sigma values near zero, smoothing does not occur. Changing sigma i
directly affects the blur effect on the image. However, sigma s does not affect blur rate. There
is no big effect on the image after changing only the sigma s. Sharpness does not necessary that
much with sigma s rather than sigma i. To have a more blurred image, we should take sigma values
bigger.
"""
```
