<template>
  <div>
    <div class="map-wrapper" v-if="uploaded">
      <q-btn class="remove-map" rounded color="primary" icon="remove" @click="removeMap" label="Remove map"/>
      <div class="map" v-html="file"></div>
    </div>
    <q-file
      v-model="file"
      v-else
      label="Pick one file"
      filled
      clearable
      style="max-width: 300px"
      accept=".svg"
      class="file-upload-wrapper"
      @rejected="onRejected"
    >
      <template v-slot:prepend>
        <q-icon name="attach_file" />
      </template>
      <template v-slot:after>
        <q-btn round dense flat icon="send" @click="sendMap"/>
      </template>
    </q-file>
  </div>
</template>

<script lang="ts" setup>
import {
  ref,
  onMounted,
} from 'vue';
import { api, axios } from 'boot/axios'

const file = ref<File | null>(null);
const uploaded = ref(false);
const TOKEN = '<TOKEN goes here>';

  const onRejected = (rejectedEntries: never[]) => {
    console.error(`${rejectedEntries.length} file(s) did not pass validation constraints`)
  };

  const sendMap = () => {
    if (!file.value){
      return
    }

    const formData = new FormData();
    formData.append('file', file.value)

    api.post('', formData, {
      headers: {
        Authorization: `FlespiToken ${TOKEN}`
      }
    });
  }

onMounted(async () => {
  const uuid = await api.get('', {
    headers: {
      Authorization: `FlespiToken ${TOKEN}`
    }
  }).then(res => (res.data.result?.[0]?.uuid ?? null));

  if (uuid) {
    file.value = await axios.get(`https://cdn.flespi.io/file/${uuid}`).then(r => r.data) as File;
    uploaded.value = true;
  }
});

const removeMap = async () => {
  const { data } = await api.delete('', {
    headers: {
      Authorization: `FlespiToken ${TOKEN}`
    }
  });

  if (data) {
    file.value = null;
    uploaded.value = false;
  }
}
</script>
<style scoped>
.file-upload-wrapper {
  .q-field--filled .q-field__control {
    background: transparent;
  }
}

.map {
  padding: 20px;
  width: 100%;
  height: auto;
}

.map-wrapper {
  position: relative;
}

.remove-map {
  position: absolute;
  top: 20px;
  right: 20px;
}
</style>
