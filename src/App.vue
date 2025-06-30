<script setup lang="ts">
import { ref, reactive, watch, watchEffect } from 'vue'
const idioma = ref<string | undefined>(undefined)
let formateador = undefined
const ajustarFormateador = () => {
	formateador = new Intl.DateTimeFormat(idioma.value ? [idioma.value] : idioma.value, { month: 'long' })
	const fecha = new Date('2000-01-01T00:00:00.000')
	
	let meses = Array(12)
		.map((i, n) => {
			fecha.setMonth(i + 1)

		})
}
watch(
	idioma,
	() => {
		if (idioma.value && idioma.value.length === 2) {
			ajustarFormateador()
		}
	}
)
ajustarFormateador()


import { parse, type Options } from 'csv-parse/browser/esm/sync'
const csvinput = ref<HTMLInputElement>()
const contenido = ref()
const leerArchivo = () => {
	if (!csvinput.value?.files || !csvinput.value.files.length) {
		return
	}
	//contenido.value = ''
	csvinput.value.files[0].text()
		.then(texto => {
			// Interpreto el CSV
			const opciones: Options = {
				skip_empty_lines: true,
				fromLine: 2,
			}
			contenido.value = parse(texto, opciones)

			// Transformo valores de fecha (columna 2) a valores más manejables

		})
}



</script>

<template>
	<div class="gap-2">
		<ol>
			<li>What language is the data CSV in? (Defaults to current locale)
				<input type="text"
					   v-model="idioma"
					   class="w-8 border-b-gray-300 border-2 rounded-2"
				/>
			</li>
			<li>Load the CSV data file:
				<input type="file" accept="text/csv" ref="csvinput" @change="leerArchivo" /></li>
		</ol>
		<div class="gap-2">
		</div>
		<div class="w-full m-2">
			<pre>{{ contenido }}</pre>
		</div>
	</div>
</template>

<style scoped>
</style>
