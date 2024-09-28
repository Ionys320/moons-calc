<script setup lang="ts">
const base = ref('');
const final = ref('');

const finalDateString = ref(new Date().toISOString().split('T')[0]);

const execute = () => {
  // Define regex
  const regexMoons = /(\d{1,2}) lune/;
  const regexDate = /(\d{2})\.(\d{2})\.(\d{4})/;

  const firstDateRegex = base.value.match(regexDate);
  if (!firstDateRegex)
    return alert('Data seems invalid');

  const finalDate = new Date(finalDateString.value);
  const firstDate = new Date(`${firstDateRegex[3]}-${firstDateRegex[2]}-${firstDateRegex[1]}`);

  // For each line starting with a "•"
  const originalLines = base.value.split('\n').filter(line => line.startsWith('•'));

  let newText = base.value;

  // We find a line containing the same date than the first one
  const firstLineWithFirstDate = originalLines.find(line => line.includes(firstDateRegex![0]));
  if (!firstLineWithFirstDate)
    return alert('Data seems invalid');

  // We use this line as a reference to perform the moon calculation
  const finalDateAndFirstDateDiff = Math.floor((finalDate.getTime() - firstDate.getTime()) / (1000 * 60 * 60 * 24));
  const moons = Math.floor(finalDateAndFirstDateDiff / 14);

  for (let i = 0; i < originalLines.length; i++) {
    let line = originalLines[i];

    // Moon
    // Extract the moon
    const moonRegex = line.match(regexMoons)![1];

    // We add the moons
    const newMoons = moons + parseInt(moonRegex);

    // We replace the moon
    line = line.replace(moonRegex, newMoons.toString());

    // Date
    // Extract the date
    const dateRegex = line.match(regexDate)!;
    const date = new Date(`${dateRegex![3]}-${dateRegex![2]}-${dateRegex![1]}`);

    // We add the days
    date.setDate(date.getDate() + finalDateAndFirstDateDiff);

    // We replace the date
    const newDateString = `${date.getDate().toString().padStart(2, '0')}.${(date.getMonth() + 1).toString().padStart(2, '0')}.${date.getFullYear()}`;
    line = line.replace(dateRegex![0], newDateString);

    // We replace the line
    newText = newText.replace(originalLines[i], line);
  }

  final.value = newText;
}
</script>

<template>
  <main class="flex flex-col gap-2 p-2 h-full">
    <h1 class="text-white text-center font-bold text-2xl">Moons calculator</h1>

    <div class="grid lg:grid-cols-2 gap-2 grow h-full">
      <textarea v-model="base" placeholder="Base" class="bg-black bg-opacity-40 text-white rounded-lg p-1" />
      <textarea v-model="final" placeholder="Final" class="bg-black bg-opacity-40 text-white rounded-lg p-1" readonly
        @click="($event.target as HTMLTextAreaElement).select()" />
    </div>

    <div class="flex gap-2">
      <label for="finalDateString" class="text-white">Final date</label>
      <input v-model="finalDateString" name="finalDateString" type="date"
        class="bg-black bg-opacity-40 text-white rounded-lg p-1" />

      <button @click="execute" class="bg-blue-500 text-white p-2 rounded-lg">Calculate</button>
    </div>
  </main>
</template>

<style>
#__nuxt {
  height: 100vh;
  background-color: #000E38;
}
</style>