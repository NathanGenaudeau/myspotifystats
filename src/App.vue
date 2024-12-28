<script setup lang="ts">
import { ref } from 'vue';

import { Chart, registerables } from 'chart.js';
import { PolarAreaChart } from 'vue-chart-3';
Chart.register(...registerables);

import dayjs from 'dayjs';
import weekday from 'dayjs/plugin/weekday';
import duration from 'dayjs/plugin/duration';
dayjs.extend(weekday);
dayjs.extend(duration);

interface Song {
  ts: string,
  username: string,
  platform: string,
  ms_played: number,
  conn_country: string,
  ip_addr_decrypted: string,
  user_agent_decrypted: string,
  master_metadata_track_name: string,
  master_metadata_album_artist_name: string,
  master_metadata_album_album_name: string,
  spotify_track_uri: string,
  episode_name: string,
  episode_show_name: string,
  reason_start: string,
  reason_end: string,
  shuffle: boolean,
  skipped: boolean,
  offline: boolean,
  offline_timestamp: string,
  incognito_mode: string,
}

const data = ref<any>();
const dataParsed = ref<Song[]>([]);

const files = ref<File[]>([]);
const tab = ref<string>();

const readAsText = async (file: any) => {
		return new Promise((resolve, _)=>{
			const fileReader = new FileReader();
			fileReader.onload = function(){
				return resolve({data: fileReader.result, name:file.name, size: file.size, type: file.type});
			}
			fileReader.readAsText(file);
		})
}

const upload = async () => {
  data.value = await Promise.all(files.value.map(f => { return readAsText(f) }));
  dataParsed.value = data.value.flatMap((r : any) => JSON.parse(r.data)).filter((song: Song) => song.ms_played >= 20000);
}

const durationParsed = (time: number) => {
  const days = Math.trunc(dayjs.duration({ seconds: time }).asDays());
  const hours = Math.trunc(dayjs.duration({ seconds: (time - dayjs.duration({ days: days }).asSeconds()) }).asHours());
  const minutes = Math.trunc(dayjs.duration({ seconds: (time - (dayjs.duration({ days: days }).asSeconds() + dayjs.duration({ hours: hours }).asSeconds())) }).asMinutes());
  const seconds = Math.trunc(dayjs.duration({ seconds: (time - (dayjs.duration({ days: days }).asSeconds() + dayjs.duration({ hours: hours }).asSeconds() + dayjs.duration({ minutes: minutes }).asSeconds())) }).asSeconds());
  
  return `${days}d, ${hours}h, ${minutes}m, ${seconds}s`;
}

const mostListenedSongList = ref<any[]>([]);
const mostListenedArtistList = ref<any[]>([]);
const mostListenedHourList = ref<any[]>([]);

const mostListenedSong = () => {
  if (mostListenedSongList.value.length !== 0) return;
  // regroupement par musique
  const list = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      if (!c.spotify_track_uri) return r;
      r[c.spotify_track_uri] = r[c.spotify_track_uri] || {id: c.spotify_track_uri, name: c.master_metadata_track_name, artist: c.master_metadata_album_artist_name, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[c.spotify_track_uri].total_time += Math.trunc(c.ms_played/1000);
      r[c.spotify_track_uri].count += 1;
      return r;
    },{}));

  list.sort((a: any, b: any) => b.count - a.count);
  mostListenedSongList.value = list;
}

const mostListenedArtist = () => {
  if (mostListenedArtistList.value.length !== 0) return;
  // regroupement par artiste
  const list = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      r[c.master_metadata_album_artist_name] = r[c.master_metadata_album_artist_name] || {artist: c.master_metadata_album_artist_name, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[c.master_metadata_album_artist_name].total_time += Math.trunc(c.ms_played/1000);
      r[c.master_metadata_album_artist_name].count += 1;
      return r;
    },{}));

  list.sort((a: any, b: any) => b.count - a.count);
  mostListenedArtistList.value = list;
}

const mostListenedAlbum = ():any[] => {
  // regroupement par album
  const list = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      r[c.master_metadata_album_album_name] = r[c.master_metadata_album_album_name] || {album: c.master_metadata_album_album_name, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[c.master_metadata_album_album_name].total_time += Math.trunc(c.ms_played/1000);
      r[c.master_metadata_album_album_name].count += 1;
      return r;
    },{}));

  list.sort((a: any, b: any) => b.count - a.count);
  return list;
}

const mostListenedHour = () => {
  if (mostListenedHourList.value.length !== 0) return;
  // regroupement par heure
  const list = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      const hour = dayjs(c.ts).hour();
      r[hour] = r[hour] || {hour, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[hour].total_time += Math.trunc(c.ms_played/1000);
      r[hour].count += 1;
      return r;
    },{}));

  list.sort((a: any, b: any) => b.total_time - a.total_time);
  mostListenedHourList.value = list;
  /*mostListenedHourList.value = list.map((hour: any) => hour.total_time);
  const total = mostListenedHourList.value.reduce((a, b) => a + b);
  mostListenedHourList.value = mostListenedHourList.value.map((hour: number) => (hour/total)*100);
  console.log(mostListenedHourList.value);*/
}

const mostListenedDay = ():any[] => {
  // regroupement par heure
  const list = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      const day = dayjs(c.ts).day();
      r[day] = r[day] || {day, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[day].total_time += Math.trunc(c.ms_played/1000);
      r[day].count += 1;
      return r;
    },{}));

  list.sort((a: any, b: any) => b.count - a.count);
  return list;
}


/*const options = {
  responsive: true,
  plugins: {
    datalabels: {
      color: 'white',
      anchor: 'end',
      align: 'end',
    },
  },
};*/
</script>

<template>
  <v-file-upload density="compact" variant="compact" title="Drag and drop Spotify history files" v-model="files" multiple clearable show-size>
    <template v-slot:item="{ props: itemProps }">
      <v-file-upload-item v-bind="itemProps" lines="one" nav>
        <template v-slot:prepend>
          <v-avatar size="32" rounded></v-avatar>
        </template>

        <template v-slot:clear="{ props: clearProps }">
          <v-btn color="primary" v-bind="clearProps"></v-btn>
        </template>
      </v-file-upload-item>
    </template>
  </v-file-upload>
  <v-btn color="green" @click="upload">Import</v-btn>

  <v-tabs
      v-model="tab"
      bg-color="primary"
    >
    <v-tab value="1" @click="mostListenedSong">Most listened-to song</v-tab>
    <v-tab value="2" @click="mostListenedArtist">Most listened-to artist</v-tab>
    <v-tab value="3">Most listened-to album</v-tab>
    <v-tab value="4" @click="mostListenedHour">Most listened-to hour</v-tab>
    <v-tab value="5">Most listened-to day</v-tab>
  </v-tabs>
  <v-card-text>
      <v-tabs-window v-model="tab">
        <v-tabs-window-item value="1">
          <v-data-table :items="mostListenedSongList">
          </v-data-table>
          <!--<div v-for="(song, index) in mostListenedSongList">
            <div>n°{{index+1}} : {{ song.name }} by {{ song.artist }} - {{ song.count }} listens - {{ durationParsed(song.total_time) }}</div>
          </div>-->
        </v-tabs-window-item>
        <v-tabs-window-item value="2">
          <div v-for="(artist, index) in mostListenedArtistList">
            <div>n°{{index+1}} : {{ artist.artist }} - listened to {{ artist.count }} times for {{ durationParsed(artist.total_time) }}</div>
          </div>
        </v-tabs-window-item>
        <v-tabs-window-item value="3">
          <div v-for="(album, index) in mostListenedAlbum()">
            <div>n°{{index+1}} : {{ album.album }} - listened {{ album.count }} songs of it for {{ durationParsed(album.total_time) }}</div>
          </div>
        </v-tabs-window-item>
        <v-tabs-window-item value="4">
          <!--<PolarAreaChart :chartData="mostListenedHourList" :options="options" :height="300"/>-->
          <div v-for="(hour, index) in mostListenedHourList">
            <div>n°{{index+1}} : {{ hour.hour }}h for {{ durationParsed(hour.total_time) }}</div>
          </div>
        </v-tabs-window-item>
        <v-tabs-window-item value="5">
          <div v-for="(day, index) in mostListenedDay()">
            <div>n°{{index+1}} : {{ dayjs().weekday(day.day).format('dddd') }} for {{ durationParsed(day.total_time) }}</div>
          </div>
        </v-tabs-window-item>
      </v-tabs-window>
    </v-card-text>
</template>

<style scoped>
</style>
