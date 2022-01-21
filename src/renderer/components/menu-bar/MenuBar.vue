<template>
  <v-container
    fluid
    class="pt-2 pb-2"
  >
    <v-row
      class="px-4 pl-5"
      :class="{ 'update-available': hasUpdate }"
    >
      <v-col cols="auto">
        <SettingsMenu />
      </v-col>
      <v-col cols="auto">
        <v-btn
          class="ma-2 ml-1 px-2 d-inline-flex align-center"
          style="height: 28px"
          @click="helpDialog = true"
        >
          <v-icon
            style="margin-right: 4px; font-size: 16px !important"
          >
            mdi-help-circle
          </v-icon>
          <span>帮助</span>
        </v-btn>
      </v-col>
      <v-spacer />
      <v-col cols="auto">
        <span
          v-if="!hasUpdate"
          class="d-inline-flex align-center fill-height mx-3"
          style="font-size: 14px"
        >现在没有更新</span>
        <v-btn
          v-else-if="filteredModUpdates.length === 0 || !availableSMMUpdate"
          class="my-2 mx-1"
          style="height: 28px"
          @click="filteredModUpdates.length > 0 ? openModUpdatesDialog() : openSMMUpdateDialog()"
        >
          <span class="mx-1">
            {{ filteredModUpdates.length > 0 ? 'Mod更新可用' : 'SMM更新可用' }}
          </span>
        </v-btn>
        <v-menu v-else>
          <template #activator="{ on, attrs }">
            <v-btn
              class="my-2 mx-1"
              style="height: 28px"
              v-bind="attrs"
              v-on="on"
            >
              <span class="mx-1">
                SMM和mod更新可用
              </span>
            </v-btn>
          </template>
          <v-card>
            <v-list class="menu">
              <v-list-item @click="openModUpdatesDialog">
                <v-list-item-action />
                <v-list-item-content>
                  <v-list-item-title>mod更新 ({{ filteredModUpdates.length }})</v-list-item-title>
                </v-list-item-content>
              </v-list-item>

              <v-divider
                inset
                class="custom"
              />

              <v-list-item @click="openSMMUpdateDialog">
                <v-list-item-action />
                <v-list-item-content>
                  <v-list-item-title>SMM 更新 ({{ smmUpdateCount }})</v-list-item-title>
                </v-list-item-content>
              </v-list-item>

              <v-divider
                inset
                class="custom"
              />
            </v-list>
          </v-card>
        </v-menu>
      </v-col>
      <v-col cols="auto">
        <UpdatesMenu
          :available-s-m-m-update="availableSMMUpdate"
          :filtered-mod-updates="filteredModUpdates"
          :show-ignored-updates.sync="showIgnoredUpdates"
          :update-check-mode.sync="updateCheckMode"
          :update-check-in-progress="updateCheckInProgress"
          @addUpdateListener="addUpdateListener"
          @openSMMUpdateDialog="openSMMUpdateDialog"
          @openModUpdatesDialog="openModUpdatesDialog"
          @checkForUpdates="manualCheckForUpdates"
        />
      </v-col>
    </v-row>
    <ModUpdatesDialog
      ref="modUpdatesDialog"
      :filtered-mod-updates="filteredModUpdates"
      :ignored-updates="ignoredUpdates"
      :is-ignored="isIgnored"
      @ignoreUpdate="ignoreUpdate"
      @unignoreUpdate="unignoreUpdate"
      @updateItem="updateItem"
      @updateAll="updateAll"
      @viewChangelog="viewChangelog"
    />
    <ChangelogDialog
      ref="changelogDialog"
      :view-changelog-update="viewChangelogUpdate"
    />
    <SMMUpdateDialog
      ref="smmUpdateDialog"
      :available-s-m-m-update="availableSMMUpdate"
      :smm-update-notes="smmUpdateNotes"
      @updateSMMNow="updateSMMNow"
    />
    <ProfileExportProgressDialog />
    <ProfileImportProgressDialog />
    <v-dialog
      v-model="helpDialog"
    >
      <v-card>
        <v-card-title>
          帮助
        </v-card-title>
        <v-card-text>
          <h3>一般故障排除</h3>
          如果某件事情的表现不符合预期，首先要尝试的是
          <a @click="clearCache">清除缓存</a><br>
          如果这不起作用， <a @click="exportDebugData">生成调试数据</a>
          并将生成的zip文件上传到modding discord，#使用mods频道提供帮助<br>
          <br>
          <v-divider />
          <br>
          <h3>X 幸福工厂的安装错误</h3>
          <h4>Epic 游戏</h4>
          首先，检查Epic是否可以启动游戏。如果您更改了游戏的位置，则
          需要让Epic更新其安装信息。为此：<br>
          1.将游戏文件夹重命名为临时文件夹<br>
          2.开始从Epic安装到您希望游戏所在的目录（原始目录）
          文件夹名称，在步骤1之前）<br>
          3. 下载完成之后，关闭 Epic<br>
          4. 从临时文件夹复制回文件，但 .egstore文件夹<br>
          5. 启动Epic并继续安装，以便发现它实际上已经安装<br>
          <br>
          <v-divider />
          <br>
          <h3>找不到幸福工厂的安装</h3>
          <h4>Epic 游戏 / Steam游戏</h4>
          确保你已经安装了游戏，Epic/Steam可以找到并启动它。<br>
          <h4>破解版？</h4>
          我们不支持盗版与破解版MOD安装，请支持正版游戏~
          <br>
          <br>
          <v-divider />
          <br>
          <h3>问题未能解决？</h3>
          可加入QQ群一起解决 <a @click="linkDownload('https://qm.qq.com/cgi-bin/qm/qr?k=mVGQRbnGqR5Y6zodbAgn-HwoVAWUMWL7&jump_from=webapi')">QQ群:125172520</a>
        </v-card-text>
        <v-card-actions>
          <v-btn
            color="primary"
            text
            @click="helpDialog = false"
          >
            关闭当前
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>
import { mapState } from 'vuex';
import gql from 'graphql-tag';
import {
  ignoreUpdate, unignoreUpdate,
} from '@/utils';
import SettingsMenu from './SettingsMenu';
import UpdatesMenu from './UpdatesMenu';
import ProfileImportProgressDialog from './dialogs/ProfileImportProgressDialog';
import ProfileExportProgressDialog from './dialogs/ProfileExportProgressDialog';
import SMMUpdateDialog from './dialogs/SMMUpdateDialog';
import ChangelogDialog from './dialogs/ChangelogDialog';
import ModUpdatesDialog from './dialogs/ModUpdatesDialog';
import { getSetting, saveSetting } from '~/settings';

const UPDATE_CHECK_INTERVAL = 5 * 60 * 1000;

export default {
  components: {
    SettingsMenu,
    UpdatesMenu,
    ModUpdatesDialog,
    ChangelogDialog,
    SMMUpdateDialog,
    ProfileExportProgressDialog,
    ProfileImportProgressDialog,
  },
  data() {
    return {
      availableSMMUpdate: null,
      modUpdates: [],
      nextCheckForUpdates: -1,
      updateCheckInProgress: false,
      viewChangelogUpdate: null,
      showIgnoredUpdates: false,
      ignoredUpdates: [],
      cachedUpdateCheckMode: 'launch',
      helpDialog: false,
    };
  },
  computed: {
    ...mapState([
      'inProgress',
      'selectedInstall',
      'selectedProfile',
    ]),
    smmUpdateNotes() {
      if (!this.availableSMMUpdate) {
        return '';
      }
      return this.availableSMMUpdate.releaseNotes.map((release) => release.note.substr(release.note.indexOf('<h2>Changelog</h2>')).replace('<h2>Changelog</h2>', `<h2>v${release.version} changelog</h2>`)).join('\n');
    },
    hasUpdate() {
      return !!this.availableSMMUpdate || this.filteredModUpdates.length > 0;
    },
    filteredModUpdates() {
      return (this.showIgnoredUpdates ? this.modUpdates : this.modUpdates.filter((update) => !this.isIgnored(update)));
    },
    updateCheckMode: {
      get() {
        return this.cachedUpdateCheckMode;
      },
      set(value) {
        saveSetting('updateCheckMode', value);
        this.cachedUpdateCheckMode = value;
      },
    },
    smmUpdateCount() {
      if (!this.availableSMMUpdate) {
        return 0;
      }
      return this.availableSMMUpdate.releaseNotes.length;
    },
  },
  watch: {
    async selectedInstall() {
      await this.checkForUpdates();
    },
    async selectedProfile() {
      await this.checkForUpdates();
    },
  },
  mounted() {
    this.ignoredUpdates = getSetting('ignoredUpdates', []);
    this.cachedUpdateCheckMode = getSetting('updateCheckMode', 'launch');

    if (this.updateCheckMode === 'launch') {
      this.$root.$once('doneLaunchUpdateCheck', () => {
        this.addUpdateListener();
      });
    } else {
      this.addUpdateListener();
    }
    this.nextCheckForUpdates = setTimeout(() => this.checkForUpdates(), UPDATE_CHECK_INTERVAL);
  },
  methods: {
    linkDownload(url) {
      window.open(url, '_blank'); // 新窗口打开外链接
    },

    openSMMUpdateDialog() {
      this.$refs.smmUpdateDialog.smmUpdateDialog = true;
    },
    openModUpdatesDialog() {
      this.$refs.modUpdatesDialog.modUpdatesDialog = true;
    },
    addUpdateListener() {
      this.$electron.ipcRenderer.on('updateAvailable', (e, updateInfo) => {
        this.availableSMMUpdate = updateInfo;
        if (this.updateCheckMode === 'ask' || this.updateCheckmode === 'launch') {
          this.$refs.smmUpdateDialog.smmUpdateDialog = true;
        }
      });
    },
    async manualCheckForUpdates() {
      await this.checkForUpdates();
      if (this.filteredModUpdates.length > 0) {
        this.openModUpdatesDialog();
      }
    },
    async checkForUpdates() {
      if (this.updateCheckInProgress) {
        return;
      }
      this.updateCheckInProgress = true;
      clearTimeout(this.nextCheckForUpdates);
      // don't check for updates while something is in progress
      while (this.inProgress.length > 0) {
        // eslint-disable-next-line no-await-in-loop
        await new Promise((res) => setTimeout(() => res(), 500));
      }
      this.$electron.ipcRenderer.send('checkForUpdates');
      const modUpdates = await Promise.all((await this.$store.state.selectedInstall.checkForUpdates()).map(async (update) => Object.assign(update, {
        name: (await this.$apollo.query({
          query: gql`
            query getModName($modReference: ModReference!) {
              mod: getModByReference(modReference: $modReference) {
                id,
                name,
              }
            }
          `,
          variables: {
            modReference: update.item,
          },
        })).data.mod?.name || update.item,
      })));
      this.updateCheckInProgress = false;
      this.modUpdates = modUpdates;
      this.nextCheckForUpdates = setTimeout(() => this.checkForUpdates(), UPDATE_CHECK_INTERVAL);
    },
    updateSMMNow() {
      this.$root.$emit('downloadUpdate');
      this.refs.smmUpdateDialog.smmUpdateDialog = false;
    },
    async updateAll() {
      await this.$store.dispatch('updateMulti', this.filteredModUpdates);
      const currentUpdates = this.filteredModUpdates;
      this.modUpdates.removeWhere((update) => currentUpdates.includes(update));
      if (this.filteredModUpdates.length === 0) {
        this.$refs.modUpdatesDialog.modUpdatesDialog = false;
      }
    },
    async updateItem(update) {
      await this.$store.dispatch('updateSingle', update);
      this.modUpdates.remove(update);
      if (this.filteredModUpdates.length === 0) {
        this.$refs.modUpdatesDialog.modUpdatesDialog = false;
      }
    },
    ignoreUpdate(update) {
      this.ignoredUpdates = ignoreUpdate(update.item, update.version);
    },
    unignoreUpdate(update) {
      this.ignoredUpdates = unignoreUpdate(update.item, update.version);
    },
    isIgnored(update) {
      return this.ignoredUpdates.some((ignoredUpdate) => ignoredUpdate.item === update.item && ignoredUpdate.version === update.version);
    },
    viewChangelog(update) {
      this.viewChangelogUpdate = update;
      this.$refs.changelogDialog.changelogDialog = true;
    },
    exportDebugData() {
      this.$root.$emit('exportDebugData');
    },
    clearCache() {
      this.$root.$emit('clearCache');
    },
  },
};

</script>

<style>
.menu.v-list {
  background-color: var(--v-backgroundMenu-base) !important;
}
.menu .custom.v-list .v-list-item__action {
  margin: 0;
}
.menu .v-icon {
  font-size: 18px !important;
}
.menu .v-list-item {
  padding-left: 10px !important;
}
.menu .v-list-item__action:first-child {
  margin-right: 0px !important;
}
.menu .custom.v-divider--inset:not(.v-divider--vertical) {
  margin-left: 30px !important;
  max-width: calc(100% - 60px) !important;
}
.menu .custom.v-divider:not(.v-divider--inset):not(.v-divider--vertical) {
  margin-left: 10px !important;
  max-width: calc(100% - 40px) !important;
}
</style>

<style scoped>
.row.update-available, .row.update-available>* {
  -webkit-animation: update-availabe-anim 2s linear infinite alternate both;
          animation: update-availabe-anim 2s linear infinite alternate both;
  color: black !important;
}
.container {
  background-color: var(--v-backgroundMenuBar-base);
}
.col {
  padding: 0;
  padding-top: 8px;
  padding-bottom: 8px;
}

@-webkit-keyframes update-availabe-anim {
  0% {
    background-color: var(--v-backgroundMenuBar-base);
  }
  100% {
    background-color: var(--v-primary-base);
  }
}
@keyframes update-availabe-anim {
  0% {
    background-color: var(--v-backgroundMenuBar-base);
  }
  100% {
    background-color: var(--v-primary-base);
  }
}

</style>
