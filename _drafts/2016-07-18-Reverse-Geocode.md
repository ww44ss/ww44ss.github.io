---
layout: post
title: Machine Learning Reverse Geocoding Accelerators
excerpt: "An exploration of high-performance reverse geocoding algorithms"
categories: blogs
tags: [sample-post, code, highlighting]
image:
  feature: so-simple-sample-image-5.jpg
  credit: Saunders
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---


##SUMMARY

Several options for reverse geo-coding (i.e. determining a specific State and County from (latitude, longitude) coordinate pairs) are explored for both performance and accuracy. Direct reverse-geo-code API calls, which take about 200msec per point, are compared to computation via "point-in-polygon", as well as machine-learning `randomForest` and `nnet` classification models.  
A Random Forest model with accuracy approaching 98% improves throughput by a factor of 10^4^ over a web-based API call. A neural network model has faster prediction times, but its accuracy was lower and modeling times were prohibitively long.

##PROBLEM STATEMENT

Reverse geo-coded GPS coordinates from, for example, a tweet or photograph, to within a specific political boundary can be a useful tool in understanding geogaphical correlations to market and potical sentiment. While reverse-look up of coordinates can be done using web-services, such as the free Google Maps API or batch geo-coders[^2] typically it takes roughly 200msec per (latitude, longitude) pair and free queries can be limited. The purpose here is to explore various approaches to substantially improve response times while meeting acceptable accuracy (~98%).

[^1]: <https://developers.google.com/maps/>
[^2]: <http://www.findlatitudeandlongitude.com/batch-reverse-geocode/>



####A Note on Performance Measurements  
Performance results are difficult communicate because they can be misinterpreted and claims overstated. In this case I standardized data collection as much as possible to ensure some level of comparability. Each measurements was taken in a standardized run framework where the data were divided into training and test data sets and then the modeling and prediction steps were individually timed. In all cases, except the API use-case, I used so-called "user-time" since user-observed throughput (points/unit time) is the metric I am tring to optimize. For the API calls I used elapsed-time measurements since these better reflect user observed through-put. 


####Reverse geo-coding performance using Google Maps API

I collected data on the reverse geocode times using the Google Maps API and the `{googleway}` package. Geo-locations are reverse coded using the following function

~~~ r
geo_data <- google_reverse_geocode(location = c(as.numeric(x["lat"]), as.numeric(x["lon"])), key = google.key.txt)
~~~

where `secret.key` is a private key generated from my Google Maps API account. 

The time for each API call is taken as the `elapsed time` as measured by the `proc.time()` function, since this reflects the usage model (processing time is relatively short, throughput is limited by data across the network). It is this network time penalty which makes calls impractical for my application. 




The data represent `r nrow(time.data)` separate API calls on `r (nrow(time.data)/400) %>% as.integer` different times of day at three different geo loactions (Portland, Bend, and Seattle). The distribution is clearly multimodal with peaks corresponding to different geo-locations. The Seattle-area had the fastest access, while Bend had the slowest.
Internet delays are the key performance limiter on data throughput. 


~~~ css
#container {
float: left;
margin: 0 -240px 0 0;
width: 100%;
}
~~~

To modify styling and highlight colors edit `/_sass/_syntax.scss`.

{% highlight css %}
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
{% endhighlight %}

{% highlight html %}
{% raw %}
<nav class="pagination" role="navigation">
    {% if page.previous %}
        <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
    {% endif %}
    {% if page.next %}
        <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
    {% endif %}
</nav><!-- /.pagination -->
{% endraw %}
{% endhighlight %}

{% highlight ruby %}
module Jekyll
  class TagIndex < Page
    def initialize(site, base, dir, tag)
      @site = site
      @base = base
      @dir = dir
      @name = 'index.html'
      self.process(@name)
      self.read_yaml(File.join(base, '_layouts'), 'tag_index.html')
      self.data['tag'] = tag
      tag_title_prefix = site.config['tag_title_prefix'] || 'Tagged: '
      tag_title_suffix = site.config['tag_title_suffix'] || '&#8211;'
      self.data['title'] = "#{tag_title_prefix}#{tag}"
      self.data['description'] = "An archive of posts tagged #{tag}."
    end
  end
end
{% endhighlight %}


### Standard Code Block

    {% raw %}
    <nav class="pagination" role="navigation">
        {% if page.previous %}
            <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
        {% endif %}
        {% if page.next %}
            <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
        {% endif %}
    </nav><!-- /.pagination -->
    {% endraw %}


### Fenced Code Blocks

~~~ css
#container {
    float: left;
    margin: 0 -240px 0 0;
    width: 100%;
}
~~~

~~~ html
{% raw %}<nav class="pagination" role="navigation">
    {% if page.previous %}
        <a href="{{ site.url }}{{ page.previous.url }}" class="btn" title="{{ page.previous.title }}">Previous article</a>
    {% endif %}
    {% if page.next %}
        <a href="{{ site.url }}{{ page.next.url }}" class="btn" title="{{ page.next.title }}">Next article</a>
    {% endif %}
</nav><!-- /.pagination -->{% endraw %}
~~~

~~~ ruby
module Jekyll
  class TagIndex < Page
    def initialize(site, base, dir, tag)
      @site = site
      @base = base
      @dir = dir
      @name = 'index.html'
      self.process(@name)
      self.read_yaml(File.join(base, '_layouts'), 'tag_index.html')
      self.data['tag'] = tag
      tag_title_prefix = site.config['tag_title_prefix'] || 'Tagged: '
      tag_title_suffix = site.config['tag_title_suffix'] || '&#8211;'
      self.data['title'] = "#{tag_title_prefix}#{tag}"
      self.data['description'] = "An archive of posts tagged #{tag}."
    end
  end
end
~~~
