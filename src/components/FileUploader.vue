<template>
  <div>
    <h3>Please choose a file</h3>
    <input type="file" @change="onFileChange" />
    <div v-if="file !== null">
      <button @click="uploadFile">Upload</button>
      <ul>
        <li v-for="(item, index) in logs" :key="index">
          <b>{{ item.meta }}</b> {{ item.message }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import http from 'axios';

export default {
  name: 'FileUploader',
  data() {
    return {
      file: null,
      logs: [],
      serverUrl: 'http://localhost:3051',
    };
  },
  methods: {
    setLog(meta, message) {
      this.logs.unshift({ meta, message });
    },
    onFileChange(e) {
      [this.file] = e.target.files;

      this.setLog('[file selected]', `Name:${this.file.name} Type:${this.file.type}`);
    },
    async getPresignedurl() {
      const meta = '[presigned-url-request]';
      try {
        this.setLog(meta, 'Requesting to server ...');

        const result = await http.post(`${this.serverUrl}/get-presigned-url`, {
          fileName: this.file.name,
          fileType: this.file.type,
        });
        const presignedUrl = result.data.url;

        this.setLog(meta, `Success: Url = ${presignedUrl}`);

        return presignedUrl;
      } catch (err) {
        console.error(err);
        this.setLog(meta, `${err}`);
        return null;
      }
    },
    async uploadFile() {
      const meta = '[uploading]';
      try {
        const presignedUrl = await this.getPresignedurl();
        if (presignedUrl) {
          this.setLog(meta, `Uploading "${this.file.name}" ...`);

          const result = await http.put(presignedUrl, this.file, {
            headers: { 'Content-Type': this.file.type },
          });
          console.log(result);

          this.setLog(meta, 'Success: File was uploaded!!');
        } else this.setLog(meta, "Errror: Couldn't get Presigned Url");
      } catch (err) {
        console.error(err);
        this.setLog(meta, `${err}`);
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 20px 0 0;
}
ul {
  list-style-type: none;
}
li {
  text-align: left;
}
</style>
