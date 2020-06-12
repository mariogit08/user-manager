<template>
  <div>
	<div class="upload-example">
		<Cropper
			classname="upload-example-cropper"
			:src="img"
		/>
		<div class="button-wrapper">
			<span class="button" @click="$refs.file.click()">
				<input type="file" ref="file" @change="uploadImage($event)" accept="image/*">
				Upload image
			</span>
		</div>
	</div>
</div>
</template>
<script>
import { Cropper } from 'vue-advanced-cropper'

export default {  
  components: {
		Cropper
  },
  props:{
    img: null
  },
  data() {
		return {
			image: null
		}
	},  
  methods: {
    onMove (moveEvent) {
      this.$emit('move', moveEvent)
    },
    onResize (resizeEvent) {
      this.$emit('resize', resizeEvent)
    },
    aspectRatios () {
      return {
          minimum: this.aspectRatio || this.minAspectRatio,
          maximum: this.aspectRatio || this.maxAspectRatio
        }
    },
    uploadImage(event) {      
			// Reference to the DOM input element
			var input = event.target;
			// Ensure that you have a file before attempting to read it
			if (input.files && input.files[0]) {
					// create a new FileReader to read this image and convert to base64 format
          var reader = new FileReader();
          
					// Define a callback function to run, when FileReader finishes its job
					reader.onload = (e) => {
							// Note: arrow function used here, so that "this.imageData" refers to the imageData of Vue component
							// Read image as base64 and set to imageData
              this.image = e.target.result;
              this.$emit('ok', this.image)
          }
					// Start the reader job - read file as a data url (base64 format)
          reader.readAsDataURL(input.files[0]);   
			}
    },
     click (){
      this.$refs.file.click()
    },
  }
}
</script>
<style lang="less" scoped>
.upload-example-cropper {
	border: solid 1px #EEE;
	height: 300px;
	width: 100%;
}

.button-wrapper {
	display: flex;
	justify-content: center;
	margin-top: 17px;
}

.button {
	color: white;
	font-size: 16px;
	padding: 10px 20px;
	background: #3fb37f;
	cursor: pointer;
	transition: background 0.5s;
}

.button:hover {
	background: #38d890;
}

.button input {
	display: none;
}
</style>
