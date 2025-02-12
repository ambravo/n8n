<template>
	<Modal
		:name="modalName"
		:eventBus="modalBus"
		:center="true"
		:closeOnPressEscape="false"
		:beforeClose="closeDialog"
		customClass="contact-prompt-modal"
		width="460px"
	>
		<template slot="header">
			<n8n-heading tag="h2" size="xlarge" color="text-dark">{{ title }}</n8n-heading>
		</template>
		<template v-slot:content>
			<div :class="$style.description">
				<n8n-text size="medium" color="text-base">{{ description }}</n8n-text>
			</div>
			<div @keyup.enter="send">
				<n8n-input v-model="email" placeholder="Your email address" />
			</div>
			<div :class="$style.disclaimer">
				<n8n-text size="small" color="text-base"
					>David from our product team will get in touch personally</n8n-text
				>
			</div>
		</template>
		<template v-slot:footer>
			<div :class="$style.footer">
				<n8n-button label="Send" float="right" @click="send" :disabled="!isEmailValid" />
			</div>
		</template>
	</Modal>
</template>

<script lang="ts">
import Vue from 'vue';
import mixins from 'vue-typed-mixins';
import { mapGetters } from 'vuex';

import { IN8nPromptResponse } from '@/Interface';
import { VALID_EMAIL_REGEX } from '@/constants';
import { workflowHelpers } from '@/components/mixins/workflowHelpers';
import Modal from './Modal.vue';

export default mixins(workflowHelpers).extend({
	components: { Modal },
	name: 'ContactPromptModal',
	props: ['modalName'],
	data() {
		return {
			email: '',
			modalBus: new Vue(),
		};
	},
	computed: {
		...mapGetters({
			promptsData: 'settings/getPromptsData',
		}),
		title(): string {
			if (this.promptsData && this.promptsData.title) {
				return this.promptsData.title;
			}

			return 'You’re a power user 💪';
		},
		description(): string {
			if (this.promptsData && this.promptsData.message) {
				return this.promptsData.message;
			}

			return 'Your experience with n8n can help us improve — for you and our entire community.';
		},
		isEmailValid(): boolean {
			return VALID_EMAIL_REGEX.test(String(this.email).toLowerCase());
		},
	},
	methods: {
		closeDialog(): void {
			this.$telemetry.track('User closed email modal', {
				instance_id: this.$store.getters.instanceId,
				email: null,
			});
			this.$store.commit('ui/closeTopModal');
		},
		async send() {
			if (this.isEmailValid) {
				const response: IN8nPromptResponse = await this.$store.dispatch(
					'settings/submitContactInfo',
					this.email,
				);

				if (response.updated) {
					this.$telemetry.track('User closed email modal', {
						instance_id: this.$store.getters.instanceId,
						email: this.email,
					});
					this.$showMessage({
						title: 'Thanks!',
						message: "It's people like you that help make n8n better",
						type: 'success',
					});
				}
				this.$store.commit('ui/closeTopModal');
			}
		},
	},
});
</script>

<style lang="scss" module>
.description {
	margin-bottom: var(--spacing-s);
}

.disclaimer {
	margin-top: var(--spacing-4xs);
}
</style>

<style lang="scss">
.dialog-wrapper {
	.contact-prompt-modal {
		.el-dialog__body {
			padding: 16px 24px 24px;
		}
	}
}
</style>
