# Uploading Movies

Finally, take the modal in the preceding section live.

## Upload the Modal Trigger

To upload the modal trigger:

1. Add a button to the navigation bar for launching the upload modal on a click:

   ```markup
   <div class="navbar-menu">
    <div class="navbar-end">
      <a class="button navbar-item" @click="showModal = !showModal">
        Upload
      </a>  
    </div>
   </div>
   ```

   Every time you click that button, it flips a `showModal` boolean. Add the property to the data model:

   ```javascript
   showModal: false,
   ```

2. Add `UploadModal` to the `App` component as a child. Start with adding the element immediately below the container `div`:

   ```markup
   <UploadModal :showModal="showModal" @handle-       
   upload="uploadToServer"></UploadModal>
   ```

   Note that you are passing down `showModal` to the component. You will learn about the `handle-upload` event later.

3. Import `UploadModal` to `App`:

   ```javascript
   import UploadModal from './components/UploadModal.vue';

   export default {
   //...
   components: {
   //...
   UploadModal
   }
   }
   ```

## Watch `showModal`

You show the modal according to the value of the `showModal` property in the `UploadModal` component:

```javascript
watch: {
  showModal(val) {
    val ? this.$refs.modal.open() : this.$refs.modal.close();
  }
},
```

Click the **Upload** button for the modal:

![](https://d2mxuefqeaa7sj.cloudfront.net/s_C4E0BB4A3CA481FA22D9AA6239D953F2B1D94D00408DB28F7AB567E3C6C4DB1A_1521628075318_Screen+Shot+2018-03-21+at+11.27.25+AM.png)

## Upload an Image or a Video

On a click of each of the **Upload** buttons on the modal, the `startUpload` method is called but with different arguments. Create that method as follows:

```javascript
methods: {
  startUpload(type) {
    cloudinary.openUploadWidget(
      { cloud_name: 'christekh', upload_preset: 'idcidr0h' },
      (error, result) => {
        console.log(error, result[0]);
        type === 'banner'
          ? (this.banner = result[0].public_id)
          : (this.trailer = result[0].public_id);
      }
    );
  },
},
```

`startUpload` launches the upload widget, which enables you to upload images or videos. After the upload, depending on which button was clicked, set the value to reflect the banner or trailer model.

Besides `cloud_name`, you must also create `upload_preset`, unsigned, to enable uploads. To do so, fill in the fields on [Cloudinary's **Upload settings** page](https://cloudinary.com/console/settings/upload) under the upload prest section.

The upload widget uses another library for modularity. Add the following code below the scripts in the `public/index.html` file:

```markup
<script src="//widget.cloudinary.com/global/all.js" type="text/javascript"></script>
```

Clicking the **Upload** button on the modal displays this widget:

![](https://d2mxuefqeaa7sj.cloudfront.net/s_C4E0BB4A3CA481FA22D9AA6239D953F2B1D94D00408DB28F7AB567E3C6C4DB1A_1521628982263_Screen+Shot+2018-03-21+at+11.42.14+AM.png)

## Generate an Upload Event

The form contains an attached `handleUpload` event, which ensures that on a click of the **Submit** button, the titles and IDs of the media content are displayed. `handleUpload` checks if those values are available before generating an event to the parent component for an upload.

Add this code to the `methods` object:

```javascript
handleUpload() {
  const data = {
    title: this.title,
    banner: this.banner,
    trailer: this.trailer
  };
  if (data.title && this.banner && this.trailer)
    this.$emit('handle-upload', data);
}
```

## Upload to the Server

Recall that the element for `UploadModal` reads like this:

```markup
<UploadModal :showModal="showModal" @handle-upload="uploadToServer"></UploadModal>
```

The `uploadToServer` method is the handler for the event generated in the child component. Create `uploadToServer` in the `methods` object:

```javascript
uploadToServer(data) {
  axios.post(this.url, data).then(res => {
    this.movies = [...this.movies, res.data];
    this.showModal = false;
  })
}
```

`uploadToServer` pushes to the server and updates the list with the new entry. It also hides the modal.

The `App` component script reads:

```javascript
import axios from 'axios';
import VideoPlayer from './components/VideoPlayer.vue';
import VideoList from './components/VideoList.vue';
import UploadModal from './components/UploadModal.vue';

export default {
  data() {
    return {
      movie: {},
      movies: [],
      showModal: false,
      url: 'https://wt-nwambachristian-gmail_com-0.run.webtask.io/server/movies'
    }
  },
  created() {
    this.cloudinaryInstance = window.cloudinary.Cloudinary.new({
      cloud_name: 'christekh',
      secure: true
    });
    axios.get(this.url)
      .then(res => {
        this.movies = res.data;
      })
  },
  methods: {
    updatePlayer(movie) {
      this.movie = movie;
    },
    uploadToServer(data) {
      axios.post(this.url, data).then(res => {
        this.movies = [...this.movies, res.data];
        this.showModal = false;
      })
    }
  },
  components: {
    VideoPlayer,
    VideoList,
    UploadModal
  }
};
```

