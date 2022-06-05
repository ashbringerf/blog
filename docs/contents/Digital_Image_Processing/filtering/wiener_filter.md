links:
	- https://homepages.inf.ed.ac.uk/rbf/CVonline/LOCAL_COPIES/VELDHUIZEN/node15.html
	- https://www.owlnet.rice.edu/~elec539/Projects99/BACH/proj2/wiener.html
- A TRUE WIENER FILTER IMPLEMENTATION FOR IMPROVING SIGNAL TO NOISE AND RESOLUTION IN ACOUSTIC IMAGES

### 核心公式

$$g=f*h+n$$
$$R=\frac{H^{*}}{|H|^{2}+Sn/Sf}$$
![[Pasted image 20220605221229.png]]
![[Pasted image 20220605221303.png]]


### 实现
```
Yf = fft2(y);
Hf = fft2(h,N,N);
Pyf = abs(Yf).^2/N^2;

% direct implementation of the regularized inverse filter, 
% when alpha = 1, it is the Wiener filter
% Gf = conj(Hf).*Pxf./(abs(Hf.^2).*Pxf+alpha*sigma^2);

% Since we don't know Pxf, the following 
% handle singular case (zero case)
sHf = Hf.*(abs(Hf)>0)+1/gamma*(abs(Hf)==0);
iHf = 1./sHf;
iHf = iHf.*(abs(Hf)*gamma>1)+gamma*abs(sHf).*iHf.*(abs(sHf)*gamma<=1);

Pyf = Pyf.*(Pyf>sigma^2)+sigma^2*(Pyf<=sigma^2);
Gf = iHf.*(Pyf-sigma^2)./(Pyf-(1-alpha)*sigma^2);

% max(max(abs(Gf).^2)) % should be equal to gamma^2
% Restorated image without denoising
eXf = Gf.*Yf;
ex = real(ifft2(eXf));

```
```

def wiener_filter(img, kernel, K):
	kernel /= np.sum(kernel)
	dummy = np.copy(img)
	dummy = fft2(dummy)
	kernel = fft2(kernel, s = img.shape)
	kernel = np.conj(kernel) / (np.abs(kernel) ** 2 + K)
	dummy = dummy * kernel
	dummy = np.abs(ifft2(dummy))
	return dummy
	


```