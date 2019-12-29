<template>
	<q-layout view="lHh Lpr lFf">
		<q-header elevated>
			<q-toolbar class="bg-accent">
				<q-btn flat dense round @click="leftDrawerOpen = !leftDrawerOpen" icon="menu" aria-label="Menu" />

				<q-toolbar-title>
					Karaoke?
				</q-toolbar-title>

				<div>Beta v{{ getVersion() }}</div>
			</q-toolbar>
		</q-header>

		<q-drawer v-model="leftDrawerOpen" show-if-above bordered content-class="bg-grey-2">
			<q-list>
				<q-item-label header>Sobre</q-item-label>
				<q-item clickable tag="a" target="_blank" href="https://alexvieira.dev">
					<q-item-section avatar>
						<q-icon name="far fa-address-card" />
					</q-item-section>
					<q-item-section>
						<q-item-label>Desenvolvido por Alex Vieira</q-item-label>
						<q-item-label caption>alexvieira.dev</q-item-label>
					</q-item-section>
				</q-item>
			</q-list>
		</q-drawer>

		<q-page-container>
			<router-view />
		</q-page-container>
	</q-layout>
</template>

<script>
export default {
	name: 'MyLayout',

	data () {
		return {
			leftDrawerOpen: false
		}
	},

	methods: {
		getVersion() {
			return process.env.APP_VERSION
		}
	}
}
</script>
