<script setup>
import { ref } from 'vue';


const urlfoto = ref('');
const supabase = useSupabaseClient();
const props = defineProps(['path']);
const emit = defineEmits(['update:path', 'upload']);

const uploadImage = async (event) => {
  const file = event.target.files[0];

  if (file) {
    const filePath = `mahasiswa/${file.name}`;

    try {
      const { data, error } = await supabase.storage
        .from('mahasiswa')
        .upload(filePath, file, {
          contentType: file.type,
        });

      if (error) {
        throw error;
      }


      const { data: publicData, error: publicError } = supabase.storage
        .from('mahasiswa')
        .getPublicUrl(filePath);

      if (publicError) {
        throw publicError;
      }

      urlfoto.value = publicData.publicUrl;
      console.log('Emitting updated path:', urlfoto.value);
      emit('update:path', urlfoto.value);
      emit('upload');
      console.log('Image uploaded successfully!', urlfoto.value);
    } catch (error) {
      console.error('Failed to upload image:', error);
    }
  }
};
</script>

<template>
  <div>
    <input type="file" @change="uploadImage" accept="image/*" class="w-full dark:bg-gray-800 py-2 rounded-md border border-gray-900" />
  </div>
</template>
