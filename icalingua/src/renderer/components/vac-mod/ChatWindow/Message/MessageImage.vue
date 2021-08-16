<template>
	<div
		ref="imageRef"
		class="vac-image-container"
		:class="{ 'vac-image-container-loading': isImageLoading || err }"
	>
		<loader
			:style="{ top: `${imageResponsive.loaderTop}px` }"
			:show="isImageLoading"
			v-if="!isHidden"
		/>
		<div
			class="vac-message-image-mod"
			:class="{
		        'vac-image-loading': isImageLoading,
		        'vac-image-err': err,
		   		'vac-el-image-loaded':!isImageLoading
			}"
			:style="{
		        'max-height': `${imageResponsive.maxHeight}px`,
		    }"
			v-if="!isHidden"
			@click="openImage"
		>
			<el-image
				:src="message.file.url"
				fit="cover"
				referrer-policy="no-referrer"
				@load="imageLoading = false"
				@error="
		          imageLoading = false;
		          err = true;
		        "
			>
				<div slot="error" class="image-slot">
					<i class="el-icon-picture-outline"></i>
				</div>
			</el-image>
		</div>
		<format-message
			:content="message.content"
			v-if="!isHidden"
			:users="roomUsers"
			:text-formatting="textFormatting"
			@open-user-tag="$emit('open-user-tag')"
		/>

		<details v-if="isHidden">
			<summary v-if="!summary" style="cursor:pointer">
				Hidden image
			</summary>
			<summary v-if="summary" style="cursor:pointer">
				<format-message
					:content="summary"
					:users="roomUsers"
					:text-formatting="textFormatting"
					@open-user-tag="$emit('open-user-tag')"
				/>
			</summary>
			<loader
				:style="{ top: `${imageResponsive.loaderTop}px` }"
				:show="isImageLoading"
			/>
			<div
				class="vac-message-image-mod"
				:class="{
		        'vac-image-loading': isImageLoading,
		        'vac-image-err': err,
		        'vac-el-image-loaded':!isImageLoading
		    }"
				:style="{
		        'max-height': `${imageResponsive.maxHeight}px`,
		    }"
				@click="openImage"
			>
				<el-image
					:src="message.file.url"
					fit="cover"
					referrer-policy="no-referrer"
					@load="imageLoading = false"
					@error="
		          imageLoading = false;
		          err = true;
		        "
				>
					<div slot="error" class="image-slot">
						<i class="el-icon-picture-outline"></i>
					</div>
				</el-image>
			</div>
		</details>
	</div>
</template>

<script>
import Loader from "../../components/Loader";
import SvgIcon from "../../components/SvgIcon";
import FormatMessage from "../../components/FormatMessage";
import {ipcRenderer} from 'electron'

export default {
	name: "MessageImage",
	components: {SvgIcon, Loader, FormatMessage},

	props: {
		currentUserId: {type: [String, Number], required: true},
		message: {type: Object, required: true},
		roomUsers: {type: Array, required: true},
		textFormatting: {type: Boolean, required: true},
		imageHover: {type: Boolean, required: true},
	},

	data() {
		return {
			imageLoading: true,
			imageResponsive: "",
			err: false,
		};
	},

	computed: {
		isImageLoading() {
			return (
				this.message.file.url.indexOf("blob:http") !== -1 || this.imageLoading
			);
		},
		isHidden() {
			return /[!！] *[Hh] *[Ii] *[Dd] *[Ee]/.test(this.message.content)
		},
		summary() {
			return this.message.content.replace(/[!！] *[Hh] *[Ii] *[Dd] *[Ee]/, '').trim()
		}
	},

	mounted() {
		this.imageResponsive = {
			maxHeight: 232,
			loaderTop: this.$refs.imageRef.clientWidth / 2,
		};
	},

	methods: {
		openImage() {
			ipcRenderer.send('openImage', this.message.file.url, false)
		}
	}
};
</script>

<style lang="scss">
@media only screen and (max-width: 950px) {
	.vac-image-container {
		max-width: -webkit-fill-available;
	}

	.vac-image-container-loading {
		min-width: -webkit-fill-available !important;
	}

	.vac-message-image-mod {
		max-width: -webkit-fill-available;

		.el-image {
			height: -webkit-fill-available;
			width: -webkit-fill-available;

			img {
				max-height: 232px;
				max-width: 250px;
			}
		}
	}
	.vac-image-err {
		//width: -webkit-fill-available !important;
	}
	.vac-image-loading {
		width: -webkit-fill-available !important;
	}
}

@media only screen and (min-width: 950px) {
	.vac-message-image-mod {
		max-width: 250px;
		.el-image {
			img {
				height: auto;
				width: auto;
			}
		}
	}
	.vac-image-loading {
		width: 250px !important;
	}
	.vac-image-container-loading {
		width: 250px !important;
	}
	.vac-image-container {
		max-width: 250px;
	}
}

.vac-image-container {
	width: fit-content;
}

.vac-image-loading {
	filter: blur(3px);
	height: 250px;
}

.vac-image-err {
	height: 250px;
	width: 250px !important;
}

.vac-message-image-mod {
	position: relative;
	background-color: var(--chat-message-bg-color-image) !important;
	max-height: 250px;
	border-radius: 4px;
	margin: 4px auto 5px;
	transition: 0.4s filter linear;
	overflow: hidden;
	width: fit-content;

	.el-image {
		vertical-align: top;
		height: -webkit-fill-available;
		width: -webkit-fill-available;
		cursor: pointer;

		img {
			max-height: 232px;
			max-width: 250px;
		}

		.image-slot {
			display: flex;
			justify-content: center;
			align-items: center;
			width: 100%;
			height: 100%;
			font-size: 30px;
			color: #909399;
		}

	}
}
</style>