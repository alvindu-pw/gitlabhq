<script>
import { mapState, mapActions, mapGetters } from 'vuex';
import tooltip from '~/vue_shared/directives/tooltip';
import Icon from '~/vue_shared/components/icon.vue';
import DeprecatedModal from '~/vue_shared/components/deprecated_modal.vue';
import LoadingButton from '~/vue_shared/components/loading_button.vue';
import CommitFilesList from './commit_sidebar/list.vue';
import EmptyState from './commit_sidebar/empty_state.vue';
import CommitMessageField from './commit_sidebar/message_field.vue';
import SuccessMessage from './commit_sidebar/success_message.vue';
import * as consts from '../stores/modules/commit/constants';
import Actions from './commit_sidebar/actions.vue';

export default {
  components: {
    DeprecatedModal,
    Icon,
    CommitFilesList,
    EmptyState,
    SuccessMessage,
    Actions,
    LoadingButton,
    CommitMessageField,
  },
  directives: {
    tooltip,
  },
  props: {
    noChangesStateSvgPath: {
      type: String,
      required: true,
    },
    committedStateSvgPath: {
      type: String,
      required: true,
    },
  },
  computed: {
    showStageUnstageArea() {
      return !!(this.someUncommitedChanges || this.lastCommitMsg || !this.unusedSeal);
    },
    someUncommitedChanges() {
      return !!(this.changedFiles.length || this.stagedFiles.length);
    },
    ...mapState(['changedFiles', 'stagedFiles', 'rightPanelCollapsed', 'lastCommitMsg', 'unusedSeal']),
    ...mapState('commit', ['commitMessage', 'submitCommitLoading']),
    ...mapGetters('commit', ['commitButtonDisabled', 'discardDraftButtonDisabled']),
  },
  methods: {
    ...mapActions('commit', [
      'updateCommitMessage',
      'discardDraft',
      'commitChanges',
      'updateCommitAction',
    ]),
    forceCreateNewBranch() {
      return this.updateCommitAction(consts.COMMIT_TO_NEW_BRANCH).then(() => this.commitChanges());
    },
  },
};
</script>

<template>
  <div
    class="multi-file-commit-panel-section"
  >
    <deprecated-modal
      id="ide-create-branch-modal"
      :primary-button-label="__('Create new branch')"
      kind="success"
      :title="__('Branch has changed')"
      @submit="forceCreateNewBranch"
    >
      <template slot="body">
        {{ __(`This branch has changed since you started editing.
          Would you like to create a new branch?`) }}
      </template>
    </deprecated-modal>
    <template
      v-if="showStageUnstageArea"
    >
      <commit-files-list
        icon-name="unstaged"
        :title="__('Unstaged')"
        :file-list="changedFiles"
        action="stageAllChanges"
        :action-btn-text="__('Stage all')"
        item-action-component="stage-button"
      />
      <commit-files-list
        icon-name="staged"
        :title="__('Staged')"
        :file-list="stagedFiles"
        action="unstageAllChanges"
        :action-btn-text="__('Unstage all')"
        item-action-component="unstage-button"
        :show-toggle="false"
        :staged-list="true"
      />
    </template>
    <empty-state
      v-if="unusedSeal"
      :no-changes-state-svg-path="noChangesStateSvgPath"
    />
    <div
      class="multi-file-commit-panel-bottom"
    >
      <form
        class="form-horizontal multi-file-commit-form"
        @submit.prevent.stop="commitChanges"
        v-if="!rightPanelCollapsed"
      >
        <success-message
          v-if="lastCommitMsg && !someUncommitedChanges"
          :committed-state-svg-path="committedStateSvgPath"
        />
        <commit-message-field
          :text="commitMessage"
          @input="updateCommitMessage"
        />
        <div class="clearfix prepend-top-15">
          <actions />
          <loading-button
            :loading="submitCommitLoading"
            :disabled="commitButtonDisabled"
            container-class="btn btn-success btn-sm pull-left"
            :label="__('Commit')"
            @click="commitChanges"
          />
          <button
            v-if="!discardDraftButtonDisabled"
            type="button"
            class="btn btn-default btn-sm pull-right"
            @click="discardDraft"
          >
            {{ __('Discard draft') }}
          </button>
        </div>
      </form>
    </div>
  </div>
</template>
