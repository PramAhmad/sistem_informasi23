<script setup>
import { ref, watch } from 'vue';

definePageMeta({
  middleware: 'auth',
});

const urlfoto = ref('');
const title = ref('');
const desc = ref('');
const alert = ref(false);
const supabase = useSupabaseClient();

const sendToDiscord = async (message) => {
  const discordWebhookURL = 'https://discord.com/api/webhooks/1285977153954906163/wC2yNaQFHXuguk6WIUgcqNI-zte3uw4moKfl7E6Us2BycXtPkIxQmKvV0z3WJeAn7hTS';

  const data = { content: message };

  try {
    const response = await fetch(discordWebhookURL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });

    if (response.ok) {
      console.log('Pesan berhasil dikirim ke Discord');
    } else {
      console.error('Gagal mengirim pesan ke Discord');
    }
  } catch (error) {
    console.error('Gagal mengirim pesan ke Discord:', error);
  }
};

const submit = async () => {
  alert.value = false;
  console.log('Form Data:', title.value, desc.value, urlfoto.value);

  const { data, error } = await supabase.from('activity').insert({
    title: title.value,
    desc: desc.value,
    image: urlfoto.value,
  });

  alert.value = true;

  if (error) {
    console.error('Error inserting data:', error);
  } else {
    const messageToDiscord = `**${title.value}**\n\n${desc.value}\n\n${urlfoto.value}`;
    sendToDiscord(messageToDiscord);
    navigateTo('/admin/activity');
  }
};

// Watch if urlfoto updates correctly
watch(urlfoto, (newValue) => {
  console.log('Updated urlfoto in parent:', newValue);
});
</script>

<template>
  <NuxtLayout name="authenticated">
    <SectionMain>
      <SectionTitleLineWithButton
        :icon="mdiBallotOutline"
        title="Tambah Activity"
        main
      />
      <CardBox>
        <form @submit.prevent="submit">
          <FormField label="Title">
            <FormControl v-model="title" placeholder="Input Title" :icon="mdiAccount" />
          </FormField>

          <FormField label="Desc" help="Your Desc">
            <FormControl v-model="desc" type="textarea" placeholder="Hos its it going?" />
          </FormField>

          <FotoActivity v-model:path="urlfoto" />

          <div>
            <BaseButtons class="mt-4">
              <button type="submit" class="py-2 px-5 bg-sky-600 rounded-md text-white hover:bg-sky-500">Tambah</button>
              <BaseButton type="reset" color="info" outline label="Reset" />
            </BaseButtons>
          </div>
        </form>
      </CardBox>
    </SectionMain>
  </NuxtLayout>
</template>
