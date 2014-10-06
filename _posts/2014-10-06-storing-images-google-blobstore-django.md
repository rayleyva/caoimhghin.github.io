---
layout: post
title:  "Storing Images in Google App Engine with Django"
date:   2014-10-06 15:19:39
categories:
---

Hosting your images in blobstore is pretty straight forward, but when you throw Django into the mix there is a plethora of
documentation out there, a lot of which is outdated or doesn't apply. My goal here is to distill it down to a very simple
yet useful demo app.

[Clone the GitHub Repo](https://github.com/caoimhghin/django-blobstore-images)

[Ask questions here](https://github.com/caoimhghin/django-blobstore-images/issues/new)


## The Major Pieces

* Post your form data to blobstore
* Store the <kbd>blob_key</kbd> and <kbd>serving_url</kbd> (Images API) in NDB
* Serve the images directly from blobstore

## Break down

In order to post the form data to blobstore, in your first view setup a temporary post URL, we'll call it <kbd>upload_url</kbd>:

{% highlight python %}
    return self.render_to_response('home.html', context={
        'upload_url': blobstore.create_upload_url(success_path='/upload-finalize'),

{% endhighlight %}

Use this upload url as the action on your html form. Once the data is posted to this URL, blobstore will post the
data minus the files to your <kbd>success_path</kbd> for you to do all the processing you need.

In your finalize handler you'll fetch the data out of the request along with any blob_info objects and save them off
for use later. The relevant bits are shown here:

{% highlight python %}
from google.appengine.api import images

class UploadFinalizeHandler(BaseView):

    def post(self):
        image_alt = self.request.POST.get('image_alt')
        image_alt = image_alt.strip()

        for upload in get_uploads(self.request, 'image'):
            blob_key = upload.key()

            info = ImageInfo(
                serving_url=images.get_serving_url(blob_key),
                alt=image_alt,
                blob_key=blob_key,
                original_filename=upload.filename,
                content_type=upload.content_type
            )
            info.put()

            return self.redirect_to('home')
{% endhighlight %}

We're using a magic method there called <kbd>get_uploads</kbd> taken from this [Django Snippet](http://appengine-cookbook.appspot.com/recipe/blobstore-get_uploads-helper-function-for-django-request/).
The other important piece here is getting the serving url. This will be a url to one of Google's servers that will apply some optimizations to the images when they are served, and also allow you
to do some manipulation on those images just by modifying that url in your template. See [Google's docs on get_serving_url](https://cloud.google.com/appengine/docs/python/images/functions#Image_get_serving_url)
for more information.

No code sample is complete with the HTML side of it. The form is intentionally kept simple here, no javascript, no validation. You'll want to add that in, of course but this make it very clear how
your image data is posted along with other form elements and how to read and store that other data along with the image.

{% highlight html %}
<form action="{{ upload_url }}" method="POST" enctype="multipart/form-data">
    <div class="panel panel-default">
        <div class="panel-body">
            <div class="form-group">
                <span><b>Image Alt Text</b></span>
                <input class="form-control" id="id_image_alt" name="image_alt" placeholder="Tags or keywords that describe your image" required="required" type="text" />
            </div>

            <div class="form-group">
                <span><b>Image</b></span>
                <input type="file" id="image" name="image" />
            </div>
            <div id="uploadPreview"></div>
        </div>
    </div>
    <div class="text-center">
        <input type="submit" class="btn btn-primary btn-lg" value="Upload Image" />
    </div>
</form>
{% endhighlight %}

### Avoid CSRF Errors

There is one additional gotcha here. The post to your finalize handler won't have a CSRF token. I've included a middleware to just disable this, but you probably want to do it on a per-view basis instead.

{% highlight python %}
class DisableCSRF(object):
    def process_request(self, request):
        setattr(request, '_dont_enforce_csrf_checks', True)

{% endhighlight %}

### The Upload URL is one time use only

In this sample application we're setting this URL in the view that renders the page. If there is an issue or the user hits the back button that upload url won't work anymore. A better way to do that is to use
an endpoint that specifically returns a new upload URL to your page in javascript and then simply set the url on the form immediately prior to posting.

### Seeing it in action

If everything went well, then your images are loading very fast and they aren't hitting your app's instances.

[See it in action here on appspot](http://qpfaseasdf.appspot.com/)