<template>
   <div id="footer" class="text-light">
      <div class="footer-left-elements">
         <ul class="footer-elements">
            <li class="footer-element">
               <i class="mdi mdi-18px mdi-database mr-1" />
               <small>{{ versionString }}</small>
            </li>
         </ul>
      </div>

      <div class="footer-right-elements">
         <ul class="footer-elements">
            <li class="footer-element footer-link" @click="openOutside('https://www.paypal.com/paypalme/fabiodistasio')">
               <i class="mdi mdi-18px mdi-coffee mr-1" />
               <small>{{ $t('word.donate') }}</small>
            </li>
            <li class="footer-element footer-link" @click="openOutside('https://github.com/antares-sql/antares/issues')">
               <i class="mdi mdi-18px mdi-bug" />
            </li>
            <li class="footer-element footer-link" @click="showSettingModal('about')">
               <i class="mdi mdi-18px mdi-information-outline" />
            </li>
         </ul>
      </div>
   </div>
</template>

<script>
import { useApplicationStore } from '@/stores/application';
import { useWorkspacesStore } from '@/stores/workspaces';
import { storeToRefs } from 'pinia';
const { shell } = require('electron');

export default {
   name: 'TheFooter',
   setup () {
      const applicationStore = useApplicationStore();
      const workspacesStore = useWorkspacesStore();

      const { getSelected: workspace } = storeToRefs(workspacesStore);

      const { appVersion, showSettingModal } = applicationStore;
      const { getWorkspace } = workspacesStore;

      return {
         appVersion,
         showSettingModal,
         workspace,
         getWorkspace
      };
   },
   computed: {
      version () {
         return this.getWorkspace(this.workspace) ? this.getWorkspace(this.workspace).version : null;
      },
      versionString () {
         if (this.version)
            return `${this.version.name} ${this.version.number} (${this.version.arch} ${this.version.os})`;
         return '';
      }
   },
   methods: {
      openOutside (link) {
         shell.openExternal(link);
      }
   }
};
</script>

<style lang="scss">
  #footer {
    height: $footer-height;
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 0.2rem;
    position: fixed;
    bottom: 0;
    left: 0;
    right: 0;

    .footer-elements {
      list-style: none;
      margin: 0;
      display: flex;
      align-items: center;

      .footer-element {
        height: $footer-height;
        display: flex;
        align-items: center;
        padding: 0 0.4rem;
        margin: 0;

        &.footer-link {
          cursor: pointer;
          transition: background 0.2s;
        }
      }
    }
  }
</style>
