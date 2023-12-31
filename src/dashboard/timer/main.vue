<i18n>
{
  "en": {
    "panelTitle": "Timer",
    "enableChanges": "Enable Changes",
    "disableChanges": "Disable Changes",
    "toggleChangesNote": "Only use this button if needed."
  },
  "ja": {
    "panelTitle": "タイマー",
    "enableChanges": "タイマーの有効化",
    "disableChanges": "タイマーの無効化",
    "toggleChangesNote": "このボタンは必要な際に限り利用してください。"
  }
}
</i18n>

<template>
  <v-app>
    <div :class="{ Disabled: disableChanges }">
      <timer-time />
      <div
        id="Controls"
        class="d-flex justify-center"
        :style="{ 'padding-top': '10px' }"
      >
        <start-button />
        <reset-button />
        <!-- Will not show if more than 1 team -->
        <template v-if="teams.length <= 1">
          <stop-button
            :info="teams[0]"
          />
          <stop-button
            :info="teams[0]"
            forfeit
          />
          <undo-button
            :info="teams[0]"
          />
        </template>
      </div>
      <!-- Will only show if more than 1 team -->
      <div
        v-if="teams.length > 1"
        :style="{ 'padding-top': '10px' }"
      >
        <team
          v-for="(team, index) in teams"
          :key="team.id"
          :info="team"
          :index="index"
        />
      </div>
    </div>
    <div
      v-if="disableChanges || tempEnable"
      :style="{ 'padding-top': '10px' }"
    >
      <v-btn
        v-if="disableChanges"
        block
        @click="disableChanges = false; tempEnable = true"
      >
        {{ $t('enableChanges') }}
      </v-btn>
      <v-btn
        v-if="tempEnable"
        block
        @click="disableChanges = true"
      >
        {{ $t('disableChanges') }}
      </v-btn>
      <div :style="{ 'margin-top': '5px' }">
        <em>{{ $t('toggleChangesNote') }}</em>
      </div>
    </div>
    <!-- Hidden toggle for testing -->
    <!--<v-switch
      v-model="disableChanges"
      inset
      hide-details
    ></v-switch>-->
  </v-app>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'vue-property-decorator';
import { TimerChangesDisabled } from '@nodecg-speedcontrol/types/schemas';
import { RunDataTeam, RunDataActiveRun } from '@nodecg-speedcontrol/types';
import TimerTime from './components/TimerTime.vue';
import StartButton from './components/StartButton.vue';
import ResetButton from './components/ResetButton.vue';
import StopButton from './components/StopButton.vue';
import UndoButton from './components/UndoButton.vue';
import Team from './components/Team.vue';
import { replicantNS } from '../_misc/replicant_store';
import { storeModule } from './store';

@Component({
  components: {
    TimerTime,
    StartButton,
    ResetButton,
    StopButton,
    UndoButton,
    Team,
  },
})
export default class extends Vue {
  @replicantNS.State((s) => s.reps.runDataActiveRun) readonly activeRun!: RunDataActiveRun;
  @replicantNS.State(
    (s) => s.reps.timerChangesDisabled,
  ) readonly timerChangesDisabled!: TimerChangesDisabled;
  tempEnable = false;

  @Watch('disableChanges')
  onDisableChangesChange(val: boolean): void {
    if (val) {
      this.tempEnable = false;
    }
  }
  @Watch('activeRun')
  onActiveRunChange(): void {
    this.tempEnable = false;
  }

  get disableChanges(): boolean {
    return this.timerChangesDisabled;
  }
  set disableChanges(val: boolean) {
    storeModule.updateDisabledToggle(val);
  }

  get teams(): RunDataTeam[] {
    return this.activeRun?.teams || [];
  }

  mounted(): void {
    if (window.frameElement?.parentElement) {
      window.frameElement.parentElement.setAttribute(
        'display-title',
        this.$t('panelTitle') as string,
      );
    }
  }
}
</script>

<style scoped>
  .Disabled {
    pointer-events: none;
    opacity: 0.5;
  }

  #Controls > * {
    flex: 1;
  }
  #Controls > *:not(:first-child) {
    margin-left: 5px;
  }
  #Controls >>> .v-btn {
    min-width: 0;
    width: 100%;
  }
</style>
