<template>
	<el-dialog
		:visible="visible"
		:before-close="closeDialog"
		:class="{'dialog-wrapper': true, [$style.center]: center, scrollable: scrollable}"
		:width="width"
		:show-close="showClose"
		:custom-class="getCustomClass()"
		:close-on-click-modal="closeOnClickModal"
		:close-on-press-escape="closeOnPressEscape"
		:style="styles"
		append-to-body
	>
		<template v-slot:title v-if="$scopedSlots.header">
			<slot name="header" v-if="!loading" />
		</template>
		<template v-slot:title v-else-if="title">
			<div :class="centerTitle ? $style.centerTitle : ''">
				<div v-if="title">
					<n8n-heading tag="h1" size="xlarge">{{title}}</n8n-heading>
				</div>
				<div v-if="subtitle" :class="$style.subtitle">
					<n8n-heading tag="h3" size="small" color="text-light">{{subtitle}}</n8n-heading>
				</div>
			</div>
		</template>
		<div class="modal-content" @keydown.stop @keydown.enter="handleEnter" @keydown.esc="closeDialog">
			<slot v-if="!loading"  name="content"/>
			<div :class="$style.loader" v-else>
				<n8n-spinner />
			</div>
		</div>
		<div v-if="!loading && $scopedSlots.footer" :class="$style.footer">
			<slot name="footer" :close="closeDialog" />
		</div>
	</el-dialog>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
	name: "Modal",
	props: {
		name: {
			type: String,
		},
		title: {
			type: String,
		},
		subtitle: {
			type: String,
		},
		eventBus: {
			type: Vue,
		},
		showClose: {
			type: Boolean,
			default: true,
		},
		loading: {
			type: Boolean,
		},
		classic: {
			type: Boolean,
		},
		beforeClose: {
			type: Function,
		},
		customClass: {
			type: String,
		},
		center: {
			type: Boolean,
		},
		width: {
			type: String,
			default: '50%',
		},
		minWidth: {
			type: String,
		},
		maxWidth: {
			type: String,
		},
		height: {
			type: String,
		},
		minHeight: {
			type: String,
		},
		maxHeight: {
			type: String,
		},
		scrollable: {
			type: Boolean,
			default: false,
		},
		centerTitle: {
			type: Boolean,
			default: false,
		},
		closeOnClickModal: {
			type: Boolean,
			default: true,
		},
		closeOnPressEscape: {
			type: Boolean,
			default: true,
		},
	},
	mounted() {
		window.addEventListener('keydown', this.onWindowKeydown);

		if (this.$props.eventBus) {
			this.$props.eventBus.$on('close', () => {
				this.closeDialog();
			});
		}

		const activeElement = document.activeElement as HTMLElement;
		if (activeElement) {
			activeElement.blur();
		}
	},
	beforeDestroy() {
		window.removeEventListener('keydown', this.onWindowKeydown);
	},
	methods: {
		onWindowKeydown(event: KeyboardEvent) {
			if (!this.isActive) {
				return;
			}

			if (event && event.keyCode === 13) {
				this.handleEnter();
			}
		},
		handleEnter() {
			if (this.isActive) {
				this.$emit('enter');
			}
		},
		closeDialog(callback?: () => void) {
			if (this.beforeClose) {
				this.beforeClose(() => {
					this.$store.commit('ui/closeTopModal');
					if (typeof callback === 'function') {
						callback();
					}
				});

				return;
			}

			this.$store.commit('ui/closeTopModal');
			if (typeof callback === 'function') {
				callback();
			}
		},
		getCustomClass() {
			let classes = this.$props.customClass || '';

			if (this.$props.classic) {
				classes = `${classes} classic`;
			}

			return classes;
		},
	},
	computed: {
		isActive(): boolean {
			return this.$store.getters['ui/isModalActive'](this.$props.name);
		},
		visible(): boolean {
			return this.$store.getters['ui/isModalOpen'](this.$props.name);
		},
		styles() {
			const styles: {[prop: string]: string} = {};
			if (this.height) {
				styles['--dialog-height'] = this.height;
			}
			if (this.minHeight) {
				styles['--dialog-min-height'] = this.minHeight;
			}
			if (this.maxHeight) {
				styles['--dialog-max-height'] = this.maxHeight;
			}
			if (this.maxWidth) {
				styles['--dialog-max-width'] = this.maxWidth;
			}
			if (this.minWidth) {
				styles['--dialog-min-width'] = this.minWidth;
			}
			return styles;
		},
	},
});
</script>

<style lang="scss">
.dialog-wrapper {
	.el-dialog {
		display: flex;
		flex-direction: column;
		max-width: var(--dialog-max-width, 80%);
		min-width: var(--dialog-min-width, 420px);
		height: var(--dialog-height);
		min-height: var(--dialog-min-height);
		max-height: var(--dialog-max-height);
	}

	.el-dialog__body {
		overflow: hidden;
		display: flex;
		flex-direction: column;
		flex-grow: 1;
	}

	.modal-content {
		overflow: hidden;
		flex-grow: 1;
	}

	&.scrollable .modal-content {
		overflow-y: auto;
	}
}
</style>

<style lang="scss" module>
.center {
		display: flex;
		align-items: center;
		justify-content: center;
}

.loader {
	display: flex;
	align-items: center;
	justify-content: center;
	color: var(--color-primary-tint-1);
	font-size: 30px;
	height: 80%;
}

.centerTitle {
	text-align: center;
}

.subtitle {
	margin-top: var(--spacing-2xs);
}

.footer {
	margin-top: var(--spacing-l);
}
</style>
