<template>
	<q-page padding>
		<q-input clearable outlined v-model="search" debounce="500" placeholder="Busque por artista ou música" type="search" class="q-mb-md" />

		<q-infinite-scroll @load="onLoad" :offset="500" ref="scroll">
			<q-list bordered separator padding>
				<q-item v-for="(song, index) in filteredSongs" :key="song.code">
					<q-item-section avatar>
						<q-btn round flat size="sm" icon="far fa-plus" @click="addToList(song)" />
					</q-item-section>

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

		<q-dialog v-model="show_list" maximized transition-show="slide-up" transition-hide="slide-down">
			<q-card>
				<q-toolbar class="bg-accent text-white">
					<q-toolbar-title>
						Minha lista
					</q-toolbar-title>

					<q-space />

					<q-btn flat round dense icon="close" v-close-popup />
				</q-toolbar>

				<q-list bordered separator padding>
					<q-item v-for="(song, index) in list" :key="song.code">
						<q-item-section avatar>
							<q-btn round flat size="sm" icon="far fa-times" @click="removeFromList(song)" />
						</q-item-section>

						<q-item-section>
							<q-item-label>{{ song.singer }}</q-item-label>
							<q-item-label lines="1">{{ song.title }}</q-item-label>
							<q-item-label caption lines="2">{{ song.lyrics }}</q-item-label>
						</q-item-section>

						<q-item-section avatar>
							<strong>{{ song.code }}</strong>
						</q-item-section>
					</q-item>

					<q-item v-if="list.length === 0">
						<q-item-section>
							<q-item-label>Nenhuma música adicionada a lista</q-item-label>
							<q-item-label caption lines="2">Você ainda não adicionou nenhuma música a sua lista</q-item-label>
						</q-item-section>
					</q-item>
				</q-list>
			</q-card>
		</q-dialog>

		<q-page-sticky position="bottom-right" :offset="[18, 18]">
			<q-fab color="purple" icon="keyboard_arrow_up" direction="up">
				<q-fab-action color="primary" @click="show_list = true" icon="far fa-list" />
				<q-fab-action color="secondary" @click="goToTop()" icon="far fa-arrow-up" />
			</q-fab>
		</q-page-sticky>
	</q-page>
</template>

<script>
import { db } from 'boot/vuefire'

export default {
	name: 'PageIndex',

	data() {
		return {
			index: 0,
			list: [],
			show_list: false,
			search: '',
			songs: [],
		}
	},

	computed: {
		filteredSongs() {
			if (this.search) {
				this.index = 0
				this.$refs.scroll.reset()

				return this.songs.filter((s) => {
					return s.singer.toString().toUpperCase().includes(this.search.toUpperCase()) || s.title.toString().toUpperCase().includes(this.search.toUpperCase())
				}).slice(0, (this.index * 20) + 20)
			}

			return this.songs.slice(0, (this.index * 20) + 20)
		}
	},

	methods: {
		onLoad(index, done) {
			this.index = index
			done()
		},

		addToList(song) {
			let on_list = this.list.find((obj) => { return obj.code === song.code })
			if (!on_list) {
				this.list.push(song)
				this.$q.localStorage.set('list', this.list)

				this.$q.notify({
					message: 'Música adicionada a sua lista',
					caption: song.title,
					color: 'accent',
					icon: 'far fa-save'
				})
			}
			else {
				this.$q.notify({
					message: 'Música já adicionada a sua lista',
					caption: song.title,
					color: 'accent',
					icon: 'far fa-list'
				})
			}
		},

		removeFromList(song) {
			this.list = this.list.filter((item) => {
				return item.code !== song.code
			})

			this.$q.localStorage.set('list', this.list)

			this.$q.notify({
				message: 'Música removida da sua lista',
				caption: song.title,
				color: 'accent',
				icon: 'far fa-trash-alt'
			})
		},

		goToTop() {
			window.scrollTo(0,0)
		}
	},

	created() {
		this.$q.loading.show()

		this.list = this.$q.localStorage.getItem('list')

		this.$rtdbBind('songs', db.ref('songs').orderByChild('singer')).then(songs => {
			this.songs === songs
		})
		.then(() => {
			this.$q.loading.hide()
		})
	}
}
</script>
