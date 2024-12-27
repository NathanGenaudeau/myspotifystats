<script setup lang="ts">
import { ref } from 'vue';

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
const list = ref<any>([]);

const readAsText = async (file: any) => {
		return new Promise((resolve, _)=>{
			const fileReader = new FileReader();
			fileReader.onload = function(){
				return resolve({data: fileReader.result, name:file.name, size: file.size, type: file.type});
			}
			fileReader.readAsText(file);
		})
}

const files = ref<Blob[]>([]);

const upload = async () => {
  data.value = await Promise.all(files.value.map(f => { return readAsText(f) }));
  dataParsed.value = data.value.flatMap((r : any) => JSON.parse(r.data)).filter((song: Song) => song.ms_played >= 20000);
}

const tab = ref<string>();

const mostListenedSong = () => {
  // regroupement par musique
  list.value = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      r[c.spotify_track_uri] = r[c.spotify_track_uri] || {id: c.spotify_track_uri, name: c.master_metadata_track_name, artist: c.master_metadata_album_artist_name, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[c.spotify_track_uri].total_time += Math.trunc(c.ms_played/1000);
      r[c.spotify_track_uri].count += 1;
      return r;
    },{}));

  list.value.sort((a: any, b: any) => b.count - a.count);
}

const mostListenedArtist = () => {
  // regroupement par artiste
  list.value = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      r[c.master_metadata_album_artist_name] = r[c.master_metadata_album_artist_name] || {artist: c.master_metadata_album_artist_name, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[c.master_metadata_album_artist_name].total_time += Math.trunc(c.ms_played/1000);
      r[c.master_metadata_album_artist_name].count += 1;
      return r;
    },{}));

  list.value.sort((a: any, b: any) => b.count - a.count);
}

const mostListenedAlbum = () => {
  // regroupement par album
  list.value = Object.values(dataParsed.value.reduce(
    (r: any, c: Song) => {
      r[c.master_metadata_album_album_name] = r[c.master_metadata_album_album_name] || {album: c.master_metadata_album_album_name, total_time: Math.trunc(c.ms_played/1000), count: 1};
      r[c.master_metadata_album_album_name].total_time += Math.trunc(c.ms_played/1000);
      r[c.master_metadata_album_album_name].count += 1;
      return r;
    },{}));

  list.value.sort((a: any, b: any) => b.count - a.count);
}
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
    <v-tab value="one" @click="mostListenedSong">Most listened-to song</v-tab>
    <v-tab value="two" @click="mostListenedArtist">Most listened-to artist</v-tab>
    <v-tab value="three" @click="mostListenedAlbum">Most listened-to album</v-tab>
  </v-tabs>
  <v-card-text>
      <v-tabs-window v-model="tab">
        <v-tabs-window-item value="one">
          <div v-if="list.length > 0" v-for="(song, index) in list">
            <div>n°{{index+1}} : {{ song.name }} by {{ song.artist }} - {{ song.count }} listens - {{ Math.trunc(song.total_time/(3600*24)) }}d{{ Math.trunc(song.total_time%(3600*24)/3600) }}h{{ Math.trunc((song.total_time%3600)/60) }}m{{ (song.total_time%3600)%60 }}s</div>
          </div>
        </v-tabs-window-item>
        <v-tabs-window-item value="two">
          <div v-if="list.length > 0" v-for="(artist, index) in list">
            <div>n°{{index+1}} : {{ artist.artist }} - listened to {{ artist.count }} times for {{ Math.trunc(artist.total_time/(3600*24)) }}d{{ Math.trunc(artist.total_time%(3600*24)/3600) }}h{{ Math.trunc((artist.total_time%3600)/60) }}m{{ (artist.total_time%3600)%60 }}s</div>
          </div>
        </v-tabs-window-item>
        <v-tabs-window-item value="three">
          <div v-if="list.length > 0" v-for="(album, index) in list">
            <div>n°{{index+1}} : {{ album.album }} - listened {{ album.count }} songs of it for {{ Math.trunc(album.total_time/(3600*24)) }}d{{ Math.trunc(album.total_time%(3600*24)/3600) }}h{{ Math.trunc((album.total_time%3600)/60) }}m{{ (album.total_time%3600)%60 }}s</div>
          </div>
        </v-tabs-window-item>
      </v-tabs-window>
    </v-card-text>
</template>

<style scoped>
</style>
