<template lang="pug">
	span.root
		span
			span.treeview-type(:type="[type.string]")
				select(v-if="edit.value" ":value"="type.string" @change.stop.prevent="onTypeChange($event.target.value)" )
					option( v-for="type of types" ":value"="type.string") {{type.string}}
				span(v-else) {{ ` [ ${ type.string } ] ` }}
			span( v-if="nesting" )
				span.treview-toggle( @click="ui.showNested = !ui.showNested" )
					span( v-if="ui.showNested" ) ▾
					span( v-else ) ▸
				span.treview-add( :active="[edit.value]" @click="edit.value && addNode($event)" ) +
		div( v-if="nesting" )
			ul.nested( v-show="ui.showNested" )
				li.node( v-for="(nested, key) in value")
					span.treview-delete( :active="[edit.value]" @click="edit.value && deleteNode(key)" ) ×
					span.key(v-if="edit.key" )
						input( ":value"="key" @change.stop.prevent="onKeyChange(key, $event.target.value)" )
					span( v-else ) {{ key }}
					span {{ ': ' }}
					span
						TreeView(
							:value="nested"
							":deep"="(deep > 0) ? deep - 1 : 0"
							":edit"="edit"
							@change="onNestedInput(key, $event)" )
		span.treeview-value( v-else :type="[type.string]")
			span(v-if="edit.value")
				input( v-if="typeIs(value, Boolean)" type="checkbox" ":value"="!!value" @change.stop.prevent="onInput($event.target.checked)" )
				input( v-else ":value"="value" @change.stop.prevent="onInput($event.target.value)" )
			span(v-else) {{ value }}
</template>

<script>
	import get from 'lodash/get'
	import set from 'lodash/set'
	import omit from 'lodash/omit'

	import TreeView from './'
	import Type from 'type-of-is'

	const nestingTypes = [
		{},
		[]
	].map(v => ({
		string: Type.string(v),
		of: Type.of(v)
	}))

	const typesForCreation = [
		{},
		[],
		'string',
		0,
		///regexp/,
		true,
		null,
		undefined
	].map(v => ({
		string: Type.string(v),
		of: Type.of(v)
	}))

	export default {
		model: {
			prop: 'value',
			event: 'change'
		},

		props: {
			value: {
				required: true
			},

			edit: {
				type: Object,
				default: () => ({
					key: true,
					value: true
				})
			},

			deep: {
				validator: value => (typeof value === 'number' && !(value < 0)),
				default: 0
			}
		},

		data() {
			return {
				ui: {
					showNested: true
				}
			}
		},

		computed: {
			type() {
				return {
					string: Type.string(this.value),
					of: Type.of(this.value)
				}
			},

			nesting() {
				return nestingTypes.some(t => t.of == this.type.of)
			},

			types() {
				return typesForCreation
			}
		},

		methods: {
			onNestedInput(key, value) {
				this.$emit('change', set(Object.assign(new this.type.of(), this.value), key, value) )
			},

			onInput(value) {
				if(this.type.of == Number) {
					value = parseInt(value)
				}
				this.$emit('change', value)
			},

			onTypeChange(type) {
				const t = this.types.find(t => (t.string == type))

				if(t)
					this.$emit('change', t.of ? ( new t.of() ) : t.of)
				else
					console.error(`Cannot find '${type}'`)
			},

			onKeyChange(from, to) {
				const value = new this.type.of()

				for(let key in this.value)
					if(key == from)
						value[to] = this.value[from]
					else
						value[key] = this.value[key]

				this.$emit('change', value)
			},

			addNode(...args) {
				let value = new this.type.of()
				if(Type.instance(value, Array))
					value = value.concat([...this.value, 'value'])
				else
					Object.assign(value, { new_key: 'value' }, this.value)

				this.ui.showNested = true
				this.$emit('change', value)
			},

			deleteNode(key) {
				let value = new this.type.of()
				if(Type.instance(value, Array))
					value = [...this.value.slice(0, key), ...this.value.slice(key + 1, this.value.length)]
				else
					Object.assign(value, omit(this.value, [key]))
				this.$emit('change', value)
			},

			typeIs(...args) {
				return Type.is(...args)
			},

			typeOf(...args) {
				return Type.is(...args)
			},

			typeString(type) {
				return Type.string(new type.of())
			}
		},

		beforeCreate() {
			this.$options.components.TreeView = TreeView
		},

		created() {
			this.ui.showNested = this.deep > 0
		}
	}
</script>

<style lang="sass" scoped>
	$red: #e44
	$green: #2e2
	$blue: #22e

	.root
		overflow: hidden

	.nested
		margin: 0
		padding: 0

	.node
		list-style-type: none
		margin: 0
		margin-left: 1rem
		padding: 0

	.treview-toggle, .treview-delete, .treview-add
		margin: 0 .5rem
		cursor: pointer

	.treeview-type
		opacity: .5

	.treeview-value
		&[type='String']
			color: $red
		&[type='Number']
			color: $blue
		&[type='Boolean']
			color: $green

	.treview-delete, .treview-add
		&:not([active='true'])
			opacity: 0.2

	.treview-delete[active='true']
		color: $red

	treview-add[active='true']
		color: $green
</style>
