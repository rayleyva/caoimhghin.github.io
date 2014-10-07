---
layout: post
title:  "Storing Images in Google App Engine with Django"
date:   2014-10-06 15:19:39
categories:
---

Hosting your images in blobstore is just the right thing to do, but it can be frustrating to get all the moving
parts working together. The goal here is to distill it down to a very simple yet useful demo app.

## Getting Started

### Clone it
[https://github.com/caoimhghin/django-blobstore-images](https://github.com/caoimhghin/django-blobstore-images)

### Install it
Install Google Cloud SDK: [https://cloud.google.com/sdk/](https://cloud.google.com/sdk/)
<pre><code>curl https://sdk.cloud.google.com | bash
</code></pre>

Install Requirements - PIL and Django 1.5 (or latest, whatever)
<pre><code>pip install -r requirements.txt
</code></pre>

### Run it
<pre><code>dev_appserver.py .
</code></pre>

## The Major Pieces
* Post your form data to blobstore
* Store the <kbd>blob_key</kbd> and <kbd>serving_url</kbd> (Images API) in NDB
* Serve the images directly from blobstore

### Break down
Setup a temporary post URL in your view, we'll call it <kbd>upload_url</kbd>. Use this url as the action of your form.

{% highlight python %}
return self.render_to_response('home.html', context={
    'upload_url': blobstore.create_upload_url(
        success_path='/upload-finalize'
    ),
{% endhighlight %}

Create a success handler that lives at <kbd>success_path</kbd>. This handler will get your post data and blob info for
each file that you uploaded.

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

We're using a magic method there called <kbd>get_uploads</kbd> taken from this [Django Snippet](http://appengine-cookbook.appspot.com/recipe/blobstore-get_uploads-helper-function-for-django-request/), which
I don't think exists anymore, but it is linked anyway.

{% highlight python %}
def get_uploads(request, field_name=None, populate_post=False):

    if hasattr(request, '__uploads') == False:
        request.META['wsgi.input'].seek(0)
        fields = cgi.FieldStorage(
            request.META['wsgi.input'], environ=request.META)

        request.__uploads = {}
        if populate_post:
            request.POST = {}

        for key in fields.keys():
            field = fields[key]
            if isinstance(field, cgi.FieldStorage) and \
                        'blob-key' in field.type_options:
                request.__uploads.setdefault(key, []).append(
                    blobstore.parse_blob_info(field))
            elif populate_post:
                request.POST[key] = field.value

    if field_name:
        try:
            return list(request.__uploads[field_name])
        except KeyError:
            return []
    else:
        results = []
        for uploads in request.__uploads.itervalues():
            results += uploads
        return results
{% endhighlight %}

Notice that we're storing a serving url. This is a valid url until you remove it, so you can call this once and store the result on your model.
This API is pretty powerful, see [Google's docs on get_serving_url](https://cloud.google.com/appengine/docs/python/images/functions#Image_get_serving_url)
for more information.

The html form is simple, standard html. Nothing special. Your file input and other inputs all get posted to the same place.

{% highlight html %}
{% raw %}
<form action="{{ upload_url }}" method="POST" enctype="multipart/form-data">
    <div class="panel panel-default">
        <div class="panel-body">
            <div class="form-group">
                <span><b>Image Alt Text</b></span>
                <input class="form-control"
                       id="id_image_alt"
                       name="image_alt"
                       placeholder="Keywords that describe your image"
                       required="required"
                       type="text" />
            </div>

            <div class="form-group">
                <span><b>Image</b></span>
                <input type="file" id="image" name="image" />
            </div>
            <div id="uploadPreview"></div>
        </div>
    </div>
    <div class="text-center">
        <input type="submit"
               class="btn btn-primary btn-lg"
               value="Upload Image" />
    </div>
</form>
{% endraw %}
{% endhighlight %}

### CSRF

The post to your finalize handler won't have a CSRF token. I've disabled it in this app with the following middleware.

{% highlight python %}
class DisableCSRF(object):
    def process_request(self, request):
        setattr(request, '_dont_enforce_csrf_checks', True)

{% endhighlight %}

### The Upload URL is one time use only

In this sample application we're setting this URL in the view that renders the page. If there is an issue or the user hits the back button that upload url won't work anymore. A better way to do that is to use
an endpoint that returns a new upload URL to your page in javascript and then set the url on the form prior to posting.

### Live demo

[See it in action here on appspot](http://qpfaseasdf.appspot.com/)

### [Questions?](https://github.com/caoimhghin/django-blobstore-images/issues/new)