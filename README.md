# Raman
用于结构的拉曼光谱计算
%matplotlib inline
import time

import numpy as np # For data manipulation
import scipy # For data manipulation
import random
import matplotlib.pyplot as plt # For doing the plots
import lmfit
from lmfit.models import GaussianModel
import rampy as rp #Charles' libraries and functions
# get the spectrum to deconvolute, with skipping header and footer comment lines from the spectrometer
inputsp = np.genfromtxt("./data/LS4.txt",skip_header=20, skip_footer=43) 

# create a new plot for showing the spectrum
plt.figure(figsize=(5,5))
plt.plot(inputsp[:,0],inputsp[:,1],'k.',markersize=1)
plt.xlabel("Raman shift, cm$^{-1}$", fontsize = 12)
plt.ylabel("Normalized intensity, a. u.", fontsize = 12)
plt.title("Fig. 1: the raw data",fontsize = 12,fontweight="bold")
bir = np.array([(860,874),(1300,1330)]) # The regions where the baseline will be fitted
y_corr, y_base = rp.baseline(inputsp[:,0],inputsp[:,1],bir,'poly',polynomial_order=3)# We fit a polynomial background.
# signal selection
lb = 867 # The lower boundary of interest
hb = 1300 # The upper boundary of interest
x = inputsp[:,0]
x_fit = x[np.where((x > lb)&(x < hb))]
y_fit = y_corr[np.where((x > lb)&(x < hb))]
ese0 = np.sqrt(abs(y_fit[:,0]))/abs(y_fit[:,0]) # the relative errors after baseline subtraction
y_fit[:,0] = y_fit[:,0]/np.amax(y_fit[:,0])*10 # normalise spectra to maximum intensity, easier to handle 
sigma = abs(ese0*y_fit[:,0]) #calculate good ese
# create a new plot for showing the spectrum
plt.figure()
plt.subplot(1,2,1)
inp = plt.plot(x,inputsp[:,1],'k-',label='Original')
corr = plt.plot(x,y_corr,'b-',label='Corrected') #we use the sample variable because it is not already normalized...
bas = plt.plot(x,y_base,'r-',label='Baseline')
plt.xlim(lb,1300)
plt.ylim(0,40000)
plt.xlabel("Raman shift, cm$^{-1}$", fontsize = 14)
plt.ylabel("Normalized intensity, a. u.", fontsize = 14)
plt.legend()
plt.title('A) Baseline removal')

plt.subplot(1,2,2)
plt.plot(x_fit,y_fit,'k.')
plt.xlabel("Raman shift, cm$^{-1}$", fontsize = 14)
plt.title('B) signal to fit')
#plt.tight_layout()
plt.suptitle('Figure 2', fontsize = 14,fontweight = 'bold')
