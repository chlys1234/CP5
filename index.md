# Computed Photography Assignment 5

### Initials
-------------
In order to image processing the picture taken with the handhled plenoptic camera, I initialized to create one image stack. The following figure is an example of an image stack.

<pre><code> I = imread('chessboard_lightfield.png');
 
for i=1:16
    for j=1:16
        L(:,:,i,j,:) = I(i:16:end,j:16:end,:);
    end
end
 
imshow(squeeze(L(:,:,1,1,:)));

</code></pre>

![image](https://user-images.githubusercontent.com/45420635/49524929-05ec1e80-f8f0-11e8-8f6f-737a95afe6ba.png)

### Sub-aperture views
-------------
As we discussed in class, by rearranging the pixels in the light field image, we can create images that correspond to views of the scene through a pinhole placed at different points on the camera aperture. (16 x 16 such images)

<pre><code> for i=1:16
    for j=1:16
        S(1+400*(i-1):400*i,1+700*(j-1):700*j,:) = L(:,:,i,j,:);
    end
end
 
imshow(S);

</code></pre>

![image](https://user-images.githubusercontent.com/45420635/49524996-23b98380-f8f0-11e8-8a59-f505174ce9f8.png)

### Refocusing and focal-stack generation
-------------
I was able to create a refocusing image with the following formula:

![image](https://user-images.githubusercontent.com/45420635/49525037-35029000-f8f0-11e8-9f7f-df985d2fc302.png)

I set the u-v coordinate axis as follows, and then I made the focal-stack by increasing the value of d from 0 to 1.4 by 0.1. Coordinate values deviating from the coordinate axis are excluded.

<pre><code> lensletSize = 16;
maxUV = (lensletSize - 1) /2;
uindex = (1:lensletSize) - 1 - maxUV;
vindex = (1:lensletSize) - 1 - maxUV;
 
</code></pre>

Specifically, the u-coordinate was set from -7.5 to 7.5 and the v coordinate was set from 7.5 to -7.5.

<pre><code> d_array = [0:0.1:1.4];
 
R_temp = zeros(400,700,3,15);
 
for D=1:15
    d = d_array(D);
    for t=1:700
        t
        for s=1:400
            for u=1:16
                for v=1:16
                    uu = round(d*uindex(u));
                    vv = round(d*vindex(v));
                    if ((s+uu > 0) && (t-vv > 0)) && ((s+uu < 401) && (t-vv < 701))
 
                        R_temp(s,t,1,D) = R_temp(s,t,1,D) + squeeze(double(L(s+uu,t-vv,u,v,1)));
                        R_temp(s,t,2,D) = R_temp(s,t,2,D) + squeeze(double(L(s+uu,t-vv,u,v,2)));
                        R_temp(s,t,3,D) = R_temp(s,t,3,D) + squeeze(double(L(s+uu,t-vv,u,v,3)));
 
                    else
 
                    end
                end
            end
        end
    end
end
 
RR = uint8(R_temp./256);

</code></pre>

####	The refocused image when the d value is 0.

![image](https://user-images.githubusercontent.com/45420635/49525158-798e2b80-f8f0-11e8-9de7-9e4dd3047fcf.png)

####	The refocused image when the d value is 0.3.

![image](https://user-images.githubusercontent.com/45420635/49525189-8f9bec00-f8f0-11e8-9db7-03f97122ed76.png)

####	The refocused image when the d value is 0.6.

![image](https://user-images.githubusercontent.com/45420635/49525197-96c2fa00-f8f0-11e8-8617-6062ae28bc08.png)

####	The refocused image when the d value is 1.0.

![image](https://user-images.githubusercontent.com/45420635/49525213-9d517180-f8f0-11e8-862f-c4fa8432b9da.png)

####	The refocused image when the d value is 1.4.

![image](https://user-images.githubusercontent.com/45420635/49525227-a3475280-f8f0-11e8-902c-0158a1e94ae6.png)

### All-focus image and depth from defocus
-------------
All-focusing image and depth map were generated through the following steps with focal-stack.

1)	For every image in the focal stack, first convert it to the XYZ colorspace, and extract the luminance channel

![image](https://user-images.githubusercontent.com/45420635/49525311-d4c01e00-f8f0-11e8-947b-dd385bad3d94.png)

2)	Create a low frequency component by blurring it with a Gaussian kernel of standard deviation

![image](https://user-images.githubusercontent.com/45420635/49525319-d8ec3b80-f8f0-11e8-9c89-991f3dbf0781.png)

3)	Compute a high frequency component by subtracting the blurry image from the original

![image](https://user-images.githubusercontent.com/45420635/49525326-ddb0ef80-f8f0-11e8-886b-071cafb33b38.png)

4)	Compute the sharpness weight by blurring the square of high frequency component with another Gaussian kernel of standard deviation

![image](https://user-images.githubusercontent.com/45420635/49525333-e1dd0d00-f8f0-11e8-8d5a-4bf136af0d9a.png)

5)	Once you have the sharpness weights, you can compute the all-focus image

![image](https://user-images.githubusercontent.com/45420635/49525343-e6a1c100-f8f0-11e8-8dc0-ee9f13c0d736.png)

6)	You can create a per-pixel depth map by using the weights to merge depth values instead of pixel intensities.

![image](https://user-images.githubusercontent.com/45420635/49525357-ec97a200-f8f0-11e8-9b04-c0f464912c09.png)

#### Refocusing image and Depth map when σ_1=0.4, σ_2=0.4.
![image](https://user-images.githubusercontent.com/45420635/49525427-0e912480-f8f1-11e8-8518-95c050d5f4fd.png)![image](https://user-images.githubusercontent.com/45420635/49525442-13ee6f00-f8f1-11e8-87d5-39cf6432a14c.png)

#### Refocusing image and Depth map when σ_1=10, σ_2=0.4.
![image](https://user-images.githubusercontent.com/45420635/49525467-2799d580-f8f1-11e8-9286-d8dc590e89d9.png)![image](https://user-images.githubusercontent.com/45420635/49525473-2cf72000-f8f1-11e8-85ad-81bada97c467.png)

#### Refocusing image and Depth map when σ_1=0.4, σ_2=10.
![image](https://user-images.githubusercontent.com/45420635/49525482-32546a80-f8f1-11e8-8575-b148522232d4.png)![image](https://user-images.githubusercontent.com/45420635/49525490-36808800-f8f1-11e8-8c24-0db21c314e9b.png)

#### Refocusing image and Depth map when σ_1=10, σ_2=10.
![image](https://user-images.githubusercontent.com/45420635/49525504-4009f000-f8f1-11e8-9ddc-0a6fcde2ee92.png)![image](https://user-images.githubusercontent.com/45420635/49525510-439d7700-f8f1-11e8-9e53-32f0611b9a2d.png)

I have experimented with assigning various values to σ_1 and σ_2  . As a result, when σ_1 is small, the all-focus image becomes clear as a whole, but artifact occurs on the far side of depth and the depth map is not drawn well. However, if you increase the σ_2 value properly, you can see that the artifact disappears in the all-focus image and the depth map can be obtained well.
