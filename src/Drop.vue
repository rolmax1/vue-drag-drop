<template>
	<div
		@dragenter="emitEvent(events.dragenter, $event)"
		@dragleave="emitEvent(events.dragleave, $event)"
		@dragover.prevent="emitEvent(events.dragover, $event)"
		@drop="emitEvent(events.drop, $event)"
	>
		<slot :transferData="transferData"></slot>
	</div>
</template>

<script>
	import { transferDataStore } from './stores';
	import { events, mimeType, mimeDelimiter, smuggleKeyMimeType } from './constants';

	export default {
		data: () => ({ dataKey: null }),
		computed: {
			events: () => events,
			transferDataStore: () => transferDataStore,
			transferData() {
				return this.transferDataStore[this.dataKey];
			},
		},
		methods: {
			emitEvent(name, nativeEvent) {
				if (name === events.drop) {
					this.dataKey = nativeEvent.dataTransfer.getData(mimeType);
				} else {
					if (! this.dataKey) {
						const type = nativeEvent.dataTransfer.types.find(
							t => t.startsWith(smuggleKeyMimeType)
						);
						if (type) {
							this.dataKey = type.split(mimeDelimiter)[1];
						}
					}
				}

				this.$emit(name, this.transferData, nativeEvent);

				// Clean up data after emitting the event.
				if ([events.dragleave, events.drop].includes(name)) {
					this.dataKey = null;
				}
			},
		},
	};
</script>
