---
layout: post
title:  How to generate random geocoordinates within a given radius using the Haversine Formula
date:   2019-02-15
---

{% raw %}
<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-MML-AM_CHTML"></script>
{% endraw %}

### Introduction

On a previous project we needed to show markers on a map to indicate there was a property, but for privacy 
reasons we wanted to show the marker on the map close to the property but not at the exact location of the property. So
we needed to generate random points within a fixed distance of the original point location. As you would expect, I did 
some Googling first but, to my surprise, the solutions I found didn't produce something that looked like a circle when 
generating many points at exactly a given radius, so they were far from good in my opinion.

In conclusion, I had to create a better solution and that's when I decided to apply...

### The Haversine Formula

<div>
  The Haversine Formula is used to calculate the spherical distance \(d\) on a sphere of radius \(R\) between two points 
  with latitude and longitude \(\left(\varphi_1, \lambda_1\right)\) and \(\left(\varphi_2, \lambda_2\right)\):
  
  $$hav \ \frac{d}{R} = hav \left(\varphi_2 - \varphi_1\right) + \cos \varphi_1 \cos \varphi_2 \ hav \left(\lambda_2 - \lambda_1\right)$$ 
  
  Where 
  
  $$hav \ \alpha = \frac{1 - \cos \alpha}{2}$$
  
  And therefore
  
  $$
    \begin{align}
    - \cos \frac{d}{R} &= - \cos \left(\varphi_2 - \varphi_1\right) + \cos \varphi_1 \cos \varphi_2 \left(1 - \cos \left(\lambda_2 - \lambda_1 \right)\right) \\
    &= - \sin \varphi_1 \sin \varphi_2 - \cos \varphi_1 \cos \varphi_2 \cos \left(\lambda_2 - \lambda_1 \right)
    \end{align}
  $$
  
  Finally
  
  $$
    d = R \arccos \left(\sin \varphi_1 \sin \varphi_2 + \cos \varphi_1 \cos \varphi_2 \cos \left(\lambda_2 - \lambda_1 \right)\right)
  $$
</div>

### Generating random points

<p>
  Instead of using the Haversine formula to find the distance \(d\) between two points \(\left(\varphi_1, \lambda_1\right)\) 
  and \(\left(\varphi_2, \lambda_2\right)\), we're going to use it to find all points \(\left(\varphi_2, \lambda_2\right)\)
  whose distance from \(\left(\varphi_1, \lambda_1\right)\) is \(d\).
</p>


<p>
  Note \(0 \lt d \lt \pi R\), \(-\frac{\pi}{2} \lt \varphi_i \le \frac{\pi}{2} \) and \(- \pi \lt \lambda_i \le \pi \). We're also
  going to define \(\Delta \varphi = \varphi_2 - \varphi_1\) and \(\Delta \lambda = \lambda_2 - \lambda_1 \) and will be 
  solving the equation for \(\Delta \varphi\) and \(\Delta \lambda\) instead. 
</p>  

<p>
  So let's start with
  
  $$
    \cos \frac{d}{R} = \sin \varphi_1 \sin \varphi_2 + \cos \varphi_1 \cos \varphi_2 \cos \left(\lambda_2 - \lambda_1 \right)
  $$
  
  We can isolate \(\Delta \lambda\)
  
  $$
    \Delta \lambda = \pm \arccos \frac{\cos \frac{d}{R} - \sin \varphi_1 \sin \varphi_2}{\cos \varphi_1 \cos \varphi_2}
  $$
  
  And arccos only takes values between -1 and 1, therefore:
  
  $$
    -1 \le \frac{\cos \frac{d}{R} - \sin \varphi_1 \sin \varphi_2}{\cos \varphi_1 \cos \varphi_2} \le 1
  $$
  
  From the right hand side inequality
  
  $$
    \begin{align}
      \cos \frac {d}{R} \le \ &\cos \varphi_1 \cos \varphi_2 + \sin \varphi_1 \sin \varphi_2 =\\
      & \cos \left(\varphi_2 - \varphi_1\right) = \cos \Delta \varphi
    \end{align}
  $$
  
  Therefore, given \(0 \lt d \lt \pi R\)
  
  $$
    - \frac{d}{R} \le \Delta \varphi \le \frac {d}{R}
  $$
  
  From the left hand side inequality
  
  $$
    \begin{align}
      \cos \frac {d}{R} \ge \ &- \cos \varphi_1 \cos \varphi_2 + \sin \varphi_1 \sin \varphi_2 =\\
      & - \cos \left(\varphi_1 + \varphi_2\right) = - \cos \left(\Delta \varphi + 2 \varphi_1 \right)
    \end{align}
  $$
  
  Hence
  
  $$
    \cos \left(\Delta \varphi + 2 \varphi_1 \right) \ge - \cos \frac {d}{R} = \cos \left(\pi + \frac {d}{R}\right)
  $$
  
  Therefore, given \(0 \lt d \lt \pi R\) and \(-\frac{\pi}{2} \lt \varphi_i \le \frac{\pi}{2} \)
  
  $$
    - \frac{d}{R} \le \Delta \varphi \le \frac {d}{R}
  $$

  <b>In conclusion</b>, We can generate points \(\left(\varphi_2, \lambda_2\right)\) whose distance from 
  \(\left(\varphi_1, \lambda_1\right)\) is \(d\) with
  
  $$
    - \frac{d}{R} \le \Delta \varphi \le \frac {d}{R} \\
    \Delta \lambda = \pm \arccos \left(\frac{\cos \frac{d}{R} - \cos \Delta \varphi}{\cos \varphi_1 \cos \left(\Delta \varphi + \varphi_1\right)} + 1\right) \\
    \varphi_2 = \varphi_1 + \Delta \varphi, \ \lambda_2 = \lambda_1 + \Delta \lambda
  $$
  
  <b>Note 1</b> We won't get results for \(\Delta \lambda\) above when \(\varphi_1 = \pm \frac{\varphi}{2}\), that is
  for the poles. 
</p>

<p>
  <b>Note 2</b> We're using spherical coordinates, but the Earth is not a perfect sphere, so this just an approximation.
</p>

### Ruby Implementation

The code below will generate a new point that's more than 100 meters away but less than 200 meters away from the 
original point. Notice we convert the input from degrees to radians and the output from radians to degrees. Also, it would
probably be a good idea to use the big decimal library to avoid floating point errors. And lastly, if this is for a web and 
you need map markers to be at the same spot across refreshes you will have to use a pseudo-random number generator.

```ruby
lat = (point[0] * Math::PI / 180).to_d
lon = (point[1] * Math::PI / 180).to_d
max_distance = 200.to_d
min_distance = 100.to_d
earth_radius = 6_371_000.to_d
distance = rand(min_distance..max_distance)

delta_lat = rand((-1 * distance)..distance) / 6_371_000
sign = rand(0..1) * 2 - 1
puts (Math.cos(distance/earth_radius) - Math.sin(lat) * Math.sin(delta_lat + lat))
puts (Math.cos(lat) * Math.cos(delta_lat + lat))
delta_lon = sign * Math.acos(
  ((Math.cos(distance/earth_radius) - Math.cos(delta_lat)) /
    (Math.cos(lat) * Math.cos(delta_lat + lat))) + 1)

[(lat + delta_lat) * 180 / Math::PI, (lon + delta_lon) * 180 / Math::PI]
```

### Examples

<p>
Points between 100 meters and 200 meters from the Eiffel Tower, Paris (France)
</p>
![eiffel-tower](/assets/img/eiffel-tower.png 'Eiffel Tower')

<p>
Points within 2km from Quito (Ecuador)
</p>
![quito](/assets/img/quito.png 'Quito')

<p>
Points at exactly 10km from the Greenwich Observatory, London (UK)
</p>
![greenwich](/assets/img/greenwich.png 'Greenwich, London')

<p>
Points at exactly 100km from La Bombonera, Buenos Aires (Argentina)
</p>
![bombonera](/assets/img/bombonera.png 'La Bombonera, Argentina')

<p>
Points at exactly 1000km from Denver (Colorado)
</p>
![denver](/assets/img/denver.png 'Denver, Colorado')
  


