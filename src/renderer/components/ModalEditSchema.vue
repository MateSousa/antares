<template>
   <Teleport to="#window-content">
      <div class="modal active">
         <a class="modal-overlay" @click.stop="closeModal" />
         <div class="modal-container p-0">
            <div class="modal-header pl-2">
               <div class="modal-title h6">
                  <div class="d-flex">
                     <i class="mdi mdi-24px mdi-database-edit mr-1" />
                     <span class="cut-text">{{ $t('message.editSchema') }}</span>
                  </div>
               </div>
               <a class="btn btn-clear c-hand" @click.stop="closeModal" />
            </div>
            <div class="modal-body pb-0">
               <div class="content">
                  <form class="form-horizontal">
                     <div class="form-group">
                        <div class="col-3">
                           <label class="form-label">{{ $t('word.name') }}</label>
                        </div>
                        <div class="col-9">
                           <input
                              ref="firstInput"
                              v-model="database.name"
                              class="form-input"
                              type="text"
                              required
                              :placeholder="$t('message.schemaName')"
                              readonly
                           >
                        </div>
                     </div>
                     <div class="form-group">
                        <div class="col-3">
                           <label class="form-label">{{ $t('word.collation') }}</label>
                        </div>
                        <div class="col-9">
                           <BaseSelect
                              v-model="database.collation"
                              class="form-select"
                              :options="collations"
                              option-label="collation"
                              option-track-by="collation"
                           />
                           <small>{{ $t('message.serverDefault') }}: {{ defaultCollation }}</small>
                        </div>
                     </div>
                  </form>
               </div>
            </div>
            <div class="modal-footer">
               <button class="btn btn-primary mr-2" @click.stop="updateSchema">
                  {{ $t('word.update') }}
               </button>
               <button class="btn btn-link" @click.stop="closeModal">
                  {{ $t('word.close') }}
               </button>
            </div>
         </div>
      </div>
   </Teleport>
</template>

<script>
import { storeToRefs } from 'pinia';
import { useNotificationsStore } from '@/stores/notifications';
import { useWorkspacesStore } from '@/stores/workspaces';
import Schema from '@/ipc-api/Schema';
import BaseSelect from '@/components/BaseSelect.vue';

export default {
   name: 'ModalEditSchema',
   components: {
      BaseSelect
   },
   props: {
      selectedSchema: String
   },
   emits: ['close'],
   setup () {
      const { addNotification } = useNotificationsStore();
      const workspacesStore = useWorkspacesStore();

      const { getSelected: selectedWorkspace } = storeToRefs(workspacesStore);

      const { getWorkspace, getDatabaseVariable } = workspacesStore;

      return {
         addNotification,
         selectedWorkspace,
         getWorkspace,
         getDatabaseVariable
      };
   },
   data () {
      return {
         database: {
            name: '',
            prevName: '',
            collation: ''
         }
      };
   },
   computed: {
      collations () {
         return this.getWorkspace(this.selectedWorkspace).collations;
      },
      defaultCollation () {
         return this.getDatabaseVariable(this.selectedWorkspace, 'collation_server').value || '';
      }
   },
   async created () {
      let actualCollation;
      try {
         const { status, response } = await Schema.getDatabaseCollation({ uid: this.selectedWorkspace, database: this.selectedSchema });

         if (status === 'success')
            actualCollation = response;

         else
            this.addNotification({ status: 'error', message: response });
      }
      catch (err) {
         this.addNotification({ status: 'error', message: err.stack });
      }

      this.database = {
         name: this.selectedSchema,
         prevName: this.selectedSchema,
         collation: actualCollation || this.defaultCollation,
         prevCollation: actualCollation || this.defaultCollation
      };

      window.addEventListener('keydown', this.onKey);

      setTimeout(() => {
         this.$refs.firstInput.focus();
      }, 20);
   },
   beforeUnmount () {
      window.removeEventListener('keydown', this.onKey);
   },
   methods: {
      async updateSchema () {
         if (this.database.collation !== this.database.prevCollation) {
            try {
               const { status, response } = await Schema.updateSchema({
                  uid: this.selectedWorkspace,
                  ...this.database
               });

               if (status === 'success')
                  this.closeModal();
               else
                  this.addNotification({ status: 'error', message: response });
            }
            catch (err) {
               this.addNotification({ status: 'error', message: err.stack });
            }
         }
         else
            this.closeModal();
      },
      closeModal () {
         this.$emit('close');
      },
      onKey (e) {
         e.stopPropagation();
         if (e.key === 'Escape')
            this.closeModal();
      }
   }
};
</script>

<style scoped>
  .modal-container {
    max-width: 360px;
  }
</style>
