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
