<template>
	<q-page padding>
		<q-input clearable outlined v-model="search" debounce="500" placeholder="Busque por artista ou música" type="search" class="q-mb-md" />

		<q-infinite-scroll @load="onLoad" :offset="50" ref="scroll">
			<q-list bordered separator padding>
				<q-item v-for="(song, index) in filteredSongs" :key="song.code">
					<q-item-section>
						<q-item-label>{{ song.singer }}</q-item-label>
						<q-item-label lines="1">{{ song.title }}</q-item-label>
						<q-item-label caption lines="2">{{ song.lyrics }}</q-item-label>
					</q-item-section>

					<q-item-section avatar>
						<strong>{{ song.code }}</strong>
					</q-item-section>
				</q-item>

				<q-item v-if="filteredSongs.length === 0 && search">
					<q-item-section>
						<q-item-label>Nenhuma música encontrada</q-item-label>
						<q-item-label caption lines="2">Não foi encontrada nenhuma música com os termos utilizados</q-item-label>
					</q-item-section>
				</q-item>
			</q-list>
		</q-infinite-scroll>
	</q-page>
</template>

<script>
import { db } from 'boot/vuefire'

export default {
	name: 'PageIndex',

	data() {
		return {
			songs: [],
			search: '',
			index: 0
		}
	},

	computed: {
		filteredSongs() {
			if (this.search) {
				this.index = 0
				this.$refs.scroll.reset()

				return this.songs.filter((s) => {
					return s.singer.toString().toUpperCase().includes(this.search.toUpperCase()) || s.title.toString().toUpperCase().includes(this.search.toUpperCase())
				}).slice(0, (this.index * 10) + 10)
			}

			return this.songs.slice(0, (this.index * 10) + 10)
		}
	},

	methods: {
		onLoad(index, done) {
			this.index = index
			done()
		}
	},

	created() {
		this.$q.loading.show()

		this.$rtdbBind('songs', db.ref('songs').orderByChild('singer')).then(songs => {
			this.songs === songs
		})
		.then(() => {
			this.$q.loading.hide()
		})
	}
}
</script>
