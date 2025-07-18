<!--
SPDX-FileCopyrightText: syuilo and misskey-project
SPDX-License-Identifier: AGPL-3.0-only
-->

<template>
<div class="lcfyofjk">
	<svg :viewBox="`0 0 ${ viewBoxX } ${ viewBoxY }`">
		<defs>
			<linearGradient :id="cpuGradientId" x1="0" x2="0" y1="1" y2="0">
				<stop offset="0%" stop-color="hsl(180, 80%, 70%)"></stop>
				<stop offset="100%" stop-color="hsl(0, 80%, 70%)"></stop>
			</linearGradient>
			<mask :id="cpuMaskId" x="0" y="0" :width="viewBoxX" :height="viewBoxY">
				<polygon
					:points="cpuPolygonPoints"
					fill="#fff"
					fill-opacity="0.5"
				/>
				<polyline
					:points="cpuPolylinePoints"
					fill="none"
					stroke="#fff"
					stroke-width="1"
				/>
				<circle
					:cx="cpuHeadX"
					:cy="cpuHeadY"
					r="1.5"
					fill="#fff"
				/>
			</mask>
		</defs>
		<rect
			x="-2" y="-2"
			:width="viewBoxX + 4" :height="viewBoxY + 4"
			:style="{ stroke: 'none', fill: `url(#${ cpuGradientId })`, mask: `url(#${ cpuMaskId })` }"
		/>
		<text x="1" y="5">CPU <tspan>{{ cpuP }}%</tspan></text>
	</svg>
	<svg :viewBox="`0 0 ${ viewBoxX } ${ viewBoxY }`">
		<defs>
			<linearGradient :id="memGradientId" x1="0" x2="0" y1="1" y2="0">
				<stop offset="0%" stop-color="hsl(180, 80%, 70%)"></stop>
				<stop offset="100%" stop-color="hsl(0, 80%, 70%)"></stop>
			</linearGradient>
			<mask :id="memMaskId" x="0" y="0" :width="viewBoxX" :height="viewBoxY">
				<polygon
					:points="memPolygonPoints"
					fill="#fff"
					fill-opacity="0.5"
				/>
				<polyline
					:points="memPolylinePoints"
					fill="none"
					stroke="#fff"
					stroke-width="1"
				/>
				<circle
					:cx="memHeadX"
					:cy="memHeadY"
					r="1.5"
					fill="#fff"
				/>
			</mask>
		</defs>
		<rect
			x="-2" y="-2"
			:width="viewBoxX + 4" :height="viewBoxY + 4"
			:style="{ stroke: 'none', fill: `url(#${ memGradientId })`, mask: `url(#${ memMaskId })` }"
		/>
		<text x="1" y="5">MEM <tspan>{{ memP }}%</tspan></text>
	</svg>
</div>
</template>

<script lang="ts" setup>
import { onMounted, onBeforeUnmount, ref } from 'vue';
import * as Misskey from 'misskey-js';
import { genId } from '@/utility/id.js';

const props = defineProps<{
	connection: Misskey.IChannelConnection<Misskey.Channels['serverStats']>,
	meta: Misskey.entities.ServerInfoResponse
}>();

const viewBoxX = ref<number>(50);
const viewBoxY = ref<number>(30);
const stats = ref<Misskey.entities.ServerStats[]>([]);
const cpuGradientId = genId();
const cpuMaskId = genId();
const memGradientId = genId();
const memMaskId = genId();
const cpuPolylinePoints = ref<string>('');
const memPolylinePoints = ref<string>('');
const cpuPolygonPoints = ref<string>('');
const memPolygonPoints = ref<string>('');
const cpuHeadX = ref<number>();
const cpuHeadY = ref<number>();
const memHeadX = ref<number>();
const memHeadY = ref<number>();
const cpuP = ref<string>('');
const memP = ref<string>('');

onMounted(() => {
	props.connection.on('stats', onStats);
	props.connection.on('statsLog', onStatsLog);
	props.connection.send('requestLog', {
		id: genId(),
		length: 50,
	});
});

onBeforeUnmount(() => {
	props.connection.off('stats', onStats);
	props.connection.off('statsLog', onStatsLog);
});

function onStats(connStats: Misskey.entities.ServerStats) {
	stats.value.push(connStats);
	if (stats.value.length > 50) stats.value.shift();

	let cpuPolylinePointsStats = stats.value.map((s, i) => [viewBoxX.value - ((stats.value.length - 1) - i), (1 - s.cpu) * viewBoxY.value]);
	let memPolylinePointsStats = stats.value.map((s, i) => [viewBoxX.value - ((stats.value.length - 1) - i), (1 - (s.mem.active / props.meta.mem.total)) * viewBoxY.value]);
	cpuPolylinePoints.value = cpuPolylinePointsStats.map(xy => `${xy[0]},${xy[1]}`).join(' ');
	memPolylinePoints.value = memPolylinePointsStats.map(xy => `${xy[0]},${xy[1]}`).join(' ');

	cpuPolygonPoints.value = `${viewBoxX.value - (stats.value.length - 1)},${viewBoxY.value} ${cpuPolylinePoints.value} ${viewBoxX.value},${viewBoxY.value}`;
	memPolygonPoints.value = `${viewBoxX.value - (stats.value.length - 1)},${viewBoxY.value} ${memPolylinePoints.value} ${viewBoxX.value},${viewBoxY.value}`;

	cpuHeadX.value = cpuPolylinePointsStats.at(-1)![0];
	cpuHeadY.value = cpuPolylinePointsStats.at(-1)![1];
	memHeadX.value = memPolylinePointsStats.at(-1)![0];
	memHeadY.value = memPolylinePointsStats.at(-1)![1];

	cpuP.value = (connStats.cpu * 100).toFixed(0);
	memP.value = (connStats.mem.active / props.meta.mem.total * 100).toFixed(0);
}

function onStatsLog(statsLog: Misskey.entities.ServerStatsLog) {
	for (const revStats of statsLog.toReversed()) {
		onStats(revStats);
	}
}
</script>

<style lang="scss" scoped>
.lcfyofjk {
	display: flex;

	> svg {
		display: block;
		padding: 10px;
		width: 50%;

		&:first-child {
			padding-right: 5px;
		}

		&:last-child {
			padding-left: 5px;
		}

		> text {
			font-size: 4.5px;
			fill: currentColor;

			> tspan {
				opacity: 0.5;
			}
		}
	}
}
</style>
