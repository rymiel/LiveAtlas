<!--
  - Copyright 2020 James Lyne
  -
  -    Licensed under the Apache License, Version 2.0 (the "License");
  -    you may not use this file except in compliance with the License.
  -    You may obtain a copy of the License at
  -
  -      http://www.apache.org/licenses/LICENSE-2.0
  -
  -    Unless required by applicable law or agreed to in writing, software
  -    distributed under the License is distributed on an "AS IS" BASIS,
  -    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  -    See the License for the specific language governing permissions and
  -    limitations under the License.
  -->

<template>
	<div class="world" v-if="world.maps.size > 0">
		<span class="world__name" aria-hidden="true">{{ world.title }}</span>
		<div class="world__maps menu">
			<template v-for="[key, map] in world.maps" :key="`${world.name}_${key}`">
				<input :id="`${name}-${world.name}-${key}`" type="radio" :name="name"
				       v-bind:value="[world.name,map.name,map.world.name]" v-model="currentMap"
				       :aria-labelledby="`${name}-${world.name}-${key}-label`">
				<label :id="`${name}-${world.name}-${key}-label`" class="map" :for="`${name}-${world.name}-${key}`" :title="`${world.title} - ${map.title}`">
					<SvgIcon :name="getMapIcon(map)"></SvgIcon>
				</label>
			</template>
		</div>
	</div>
</template>

<script lang="ts">
import {useStore} from "@/store";
import {defineComponent} from 'vue';
import {MutationTypes} from "@/store/mutation-types";
import SvgIcon from "@/components/SvgIcon.vue";
import "@/assets/icons/block_world_surface.svg";
import "@/assets/icons/block_world_cave.svg";
import "@/assets/icons/block_world_biome.svg";
import "@/assets/icons/block_world_flat.svg";
import "@/assets/icons/block_nether_flat.svg";
import "@/assets/icons/block_nether_surface.svg";
import "@/assets/icons/block_the_end_flat.svg";
import "@/assets/icons/block_the_end_surface.svg";
import "@/assets/icons/block_other.svg";
import "@/assets/icons/block_other_flat.svg";
import "@/assets/icons/block_skylands.svg";
import {LiveAtlasWorld, LiveAtlasWorldMap} from "@/index";

export default defineComponent({
	name: 'WorldListItem',
	components: {SvgIcon},
	props: {
		world: {
			type: Object as () => LiveAtlasWorld,
			required: true
		},
		name: {
			type: String,
			default: 'map',
		}
	},

	computed: {
		currentMap: {
			get() {
				const store = useStore();
				return store.state.currentMap ? [store.state.currentWorld!.name, store.state.currentMap.name, store.state.currentMap.world.name] : undefined;
			},
			set(value: string[]) {
				useStore().commit(MutationTypes.SET_CURRENT_MAP, {worldName: value[0], mapName: value[1], realWorldName: value[2]});
			}
		}
	},

	methods: {
		getMapIcon(map: LiveAtlasWorldMap): string {
			let worldType: string,
				mapType: string;

			if (/(^|_)nether(_|$)/i.test(this.world.name) || (this.world.name == 'DIM-1') || /Nether$/.test(this.world.name) || /__(.+_)?nether(_|$)/i.test(map.name)) {
				worldType = 'nether';
				mapType = ['surface', 'nether'].includes(map.name) ? 'surface' : 'flat';
			} else if (/(^|_)end(_|$)/i.test(this.world.name) || (this.world.name == 'DIM1')) {
				worldType = 'the_end';
				mapType = ['surface', 'the_end'].includes(map.name) ? 'surface' : 'flat';
			} else {
				worldType = 'world';
				mapType = ['surface', 'flat', 'biome', 'cave'].includes(map.name) ? map.name : 'flat';
			}
			return `block_${worldType}_${mapType}`;
		}
	}
});
</script>

<style lang="scss" scoped>
	.world {
		display: flex;
		align-items: center;
		margin-bottom:  .5rem;
		padding-left: 0.8rem;

		.world__name {
			word-break: break-word;
			overflow-wrap: break-word;
		}

		.world__maps {
			display: flex;
			align-items: center;
			margin-left: auto;
			padding-left: 1rem;
			padding-right: 0.2rem;
			list-style: none;
		}
	}

	.map {
		width: 3.2rem;
		height: 3.2rem;

		.svg-icon {
			top: 0.2rem !important;
			right: 0.2rem !important;
			bottom: 0.2rem !important;
			left: 0.2rem !important;
			width: calc(100% - 0.4rem) !important;
			height: auto !important;
		}

		& ~ .map {
			margin-left: 0.5rem;
		}
	}
</style>
