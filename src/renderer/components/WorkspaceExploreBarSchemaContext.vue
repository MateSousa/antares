<template>
   <BaseContextMenu
      :context-event="contextEvent"
      @close-context="closeContext"
   >
      <div class="context-element">
         <span class="d-flex"><i class="mdi mdi-18px mdi-plus text-light pr-1" /> {{ $t('word.add') }}</span>
         <i class="mdi mdi-18px mdi-chevron-right text-light pl-1" />
         <div class="context-submenu">
            <div
               v-if="workspace.customizations.tableAdd"
               class="context-element"
               @click="openCreateTableTab"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-table text-light pr-1" /> {{ $t('word.table') }}</span>
            </div>
            <div
               v-if="workspace.customizations.viewAdd"
               class="context-element"
               @click="openCreateViewTab"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-table-eye text-light pr-1" /> {{ $t('word.view') }}</span>
            </div>
            <div
               v-if="workspace.customizations.triggerAdd"
               class="context-element"
               @click="openCreateTriggerTab"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-table-cog text-light pr-1" /> {{ $tc('word.trigger', 1) }}</span>
            </div>
            <div
               v-if="workspace.customizations.routineAdd"
               class="context-element"
               @click="openCreateRoutineTab"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-sync-circle pr-1" /> {{ $tc('word.storedRoutine', 1) }}</span>
            </div>
            <div
               v-if="workspace.customizations.functionAdd"
               class="context-element"
               @click="openCreateFunctionTab"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-arrow-right-bold-box pr-1" /> {{ $tc('word.function', 1) }}</span>
            </div>
            <div
               v-if="workspace.customizations.triggerFunctionAdd"
               class="context-element"
               @click="openCreateTriggerFunctionTab"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-cog-clockwise pr-1" /> {{ $tc('word.triggerFunction', 1) }}</span>
            </div>
            <div
               v-if="workspace.customizations.schedulerAdd"
               class="context-element"
               @click="openCreateSchedulerTab"
            >
               <span class="d-flex"><i class="mdi mdi-18px mdi-calendar-clock text-light pr-1" /> {{ $tc('word.scheduler', 1) }}</span>
            </div>
         </div>
      </div>
      <div
         v-if="workspace.customizations.schemaExport"
         class="context-element"
         @click="showExportSchemaModal"
      >
         <span class="d-flex"><i class="mdi mdi-18px mdi-database-arrow-down text-light pr-1" /> {{ $t('word.export') }}</span>
      </div>
      <div
         v-if="workspace.customizations.schemaImport"
         class="context-element"
         @click="initImport"
      >
         <span class="d-flex"><i class="mdi mdi-18px mdi-database-arrow-up text-light pr-1" /> {{ $t('word.import') }}</span>
      </div>
      <div
         v-if="workspace.customizations.schemaEdit"
         class="context-element"
         @click="showEditModal"
      >
         <span class="d-flex"><i class="mdi mdi-18px mdi-database-edit text-light pr-1" /> {{ $t('word.edit') }}</span>
      </div>
      <div
         v-if="workspace.customizations.schemaDrop"
         class="context-element"
         @click="showDeleteModal"
      >
         <span class="d-flex"><i class="mdi mdi-18px mdi-database-remove text-light pr-1" /> {{ $t('word.delete') }}</span>
      </div>

      <ConfirmModal
         v-if="isDeleteModal"
         @confirm="deleteSchema"
         @hide="hideDeleteModal"
      >
         <template #header>
            <div class="d-flex">
               <i class="mdi mdi-24px mdi-database-remove mr-1" />
               <span class="cut-text">{{ $t('message.deleteSchema') }}</span>
            </div>
         </template>
         <template #body>
            <div class="mb-2">
               {{ $t('message.deleteCorfirm') }} "<b>{{ selectedSchema }}</b>"?
            </div>
         </template>
      </ConfirmModal>
      <ModalEditSchema
         v-if="isEditModal"
         :selected-schema="selectedSchema"
         @close="hideEditModal"
      />
      <ModalExportSchema
         v-if="isExportSchemaModal"
         :selected-schema="selectedSchema"
         @close="hideExportSchemaModal"
      />
      <ModalImportSchema
         v-if="isImportSchemaModal"
         ref="importModalRef"
         :selected-schema="selectedSchema"
         @close="hideImportSchemaModal"
      />
   </BaseContextMenu>
</template>

<script>
import { useNotificationsStore } from '@/stores/notifications';
import { useWorkspacesStore } from '@/stores/workspaces';
import BaseContextMenu from '@/components/BaseContextMenu';
import ConfirmModal from '@/components/BaseConfirmModal';
import ModalEditSchema from '@/components/ModalEditSchema';
import ModalExportSchema from '@/components/ModalExportSchema';
import ModalImportSchema from '@/components/ModalImportSchema';
import Schema from '@/ipc-api/Schema';
import Application from '@/ipc-api/Application';
import { storeToRefs } from 'pinia';

export default {
   name: 'WorkspaceExploreBarSchemaContext',
   components: {
      BaseContextMenu,
      ConfirmModal,
      ModalEditSchema,
      ModalExportSchema,
      ModalImportSchema
   },
   props: {
      contextEvent: MouseEvent,
      selectedSchema: String
   },
   emits: [
      'open-create-table-tab',
      'open-create-view-tab',
      'open-create-trigger-tab',
      'open-create-routine-tab',
      'open-create-function-tab',
      'open-create-trigger-function-tab',
      'open-create-scheduler-tab',
      'close-context',
      'reload'
   ],
   setup () {
      const { addNotification } = useNotificationsStore();
      const workspacesStore = useWorkspacesStore();

      const { getSelected: selectedWorkspace } = storeToRefs(workspacesStore);

      const {
         getWorkspace,
         changeBreadcrumbs
      } = workspacesStore;

      return {
         addNotification,
         selectedWorkspace,
         getWorkspace,
         changeBreadcrumbs
      };
   },
   data () {
      return {
         isDeleteModal: false,
         isEditModal: false,
         isExportSchemaModal: false,
         isImportSchemaModal: false
      };
   },
   computed: {
      workspace () {
         return this.getWorkspace(this.selectedWorkspace);
      }
   },
   methods: {
      openCreateTableTab () {
         this.$emit('open-create-table-tab');
      },
      openCreateViewTab () {
         this.$emit('open-create-view-tab');
      },
      openCreateTriggerTab () {
         this.$emit('open-create-trigger-tab');
      },
      openCreateRoutineTab () {
         this.$emit('open-create-routine-tab');
      },
      openCreateFunctionTab () {
         this.$emit('open-create-function-tab');
      },
      openCreateTriggerFunctionTab () {
         this.$emit('open-create-trigger-function-tab');
      },
      openCreateSchedulerTab () {
         this.$emit('open-create-scheduler-tab');
      },
      showDeleteModal () {
         this.isDeleteModal = true;
      },
      hideDeleteModal () {
         this.isDeleteModal = false;
      },
      showEditModal () {
         this.isEditModal = true;
      },
      hideEditModal () {
         this.isEditModal = false;
         this.closeContext();
      },
      showExportSchemaModal () {
         this.isExportSchemaModal = true;
      },
      hideExportSchemaModal () {
         this.isExportSchemaModal = false;
         this.closeContext();
      },
      showImportSchemaModal () {
         this.isImportSchemaModal = true;
      },
      hideImportSchemaModal () {
         this.isImportSchemaModal = false;
         this.closeContext();
      },
      async initImport () {
         const result = await Application.showOpenDialog({ properties: ['openFile'], filters: [{ name: 'SQL', extensions: ['sql'] }] });
         if (result && !result.canceled) {
            const file = result.filePaths[0];
            this.showImportSchemaModal();
            this.$nextTick(() => {
               this.$refs.importModalRef.startImport(file);
            });
         }
      },
      closeContext () {
         this.$emit('close-context');
      },
      async deleteSchema () {
         try {
            const { status, response } = await Schema.deleteSchema({
               uid: this.selectedWorkspace,
               database: this.selectedSchema
            });

            if (status === 'success') {
               if (this.selectedSchema === this.workspace.breadcrumbs.schema)
                  this.changeBreadcrumbs({ schema: null });

               this.closeContext();
               this.$emit('reload');
            }
            else
               this.addNotification({ status: 'error', message: response });
         }
         catch (err) {
            this.addNotification({ status: 'error', message: err.stack });
         }
      }
   }
};
</script>
<style lang="scss" scoped>
.context-submenu {
  min-width: 150px !important;
}
</style>
