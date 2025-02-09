<template>
   <div v-show="isSelected" class="workspace-query-tab column col-12 columns col-gapless">
      <div class="workspace-query-runner column col-12">
         <div class="workspace-query-runner-footer">
            <div class="workspace-query-buttons">
               <button
                  class="btn btn-primary btn-sm"
                  :disabled="!isChanged"
                  :class="{'loading':isSaving}"
                  title="CTRL+S"
                  @click="saveChanges"
               >
                  <i class="mdi mdi-24px mdi-content-save mr-1" />
                  <span>{{ $t('word.save') }}</span>
               </button>
               <button
                  :disabled="!isChanged"
                  class="btn btn-link btn-sm mr-0"
                  :title="$t('message.clearChanges')"
                  @click="clearChanges"
               >
                  <i class="mdi mdi-24px mdi-delete-sweep mr-1" />
                  <span>{{ $t('word.clear') }}</span>
               </button>

               <div class="divider-vert py-3" />

               <button
                  class="btn btn-dark btn-sm"
                  :disabled="isChanged"
                  @click="runFunctionCheck"
               >
                  <i class="mdi mdi-24px mdi-play mr-1" />
                  <span>{{ $t('word.run') }}</span>
               </button>
               <button class="btn btn-dark btn-sm" @click="showParamsModal">
                  <i class="mdi mdi-24px mdi-dots-horizontal mr-1" />
                  <span>{{ $t('word.parameters') }}</span>
               </button>
            </div>
            <div class="workspace-query-info">
               <div class="d-flex" :title="$t('word.schema')">
                  <i class="mdi mdi-18px mdi-database mr-1" /><b>{{ schema }}</b>
               </div>
            </div>
         </div>
      </div>
      <div class="container">
         <div class="columns">
            <div class="column col-auto">
               <div class="form-group">
                  <label class="form-label">
                     {{ $t('word.name') }}
                  </label>
                  <input
                     ref="firstInput"
                     v-model="localFunction.name"
                     class="form-input"
                     :class="{'is-error': !isTableNameValid}"
                     type="text"
                  >
               </div>
            </div>
            <div v-if="customizations.languages" class="column col-auto">
               <div class="form-group">
                  <label class="form-label">
                     {{ $t('word.language') }}
                  </label>
                  <BaseSelect
                     v-model="localFunction.language"
                     :options="customizations.languages"
                     class="form-select"
                  />
               </div>
            </div>
            <div v-if="customizations.definer" class="column col-auto">
               <div class="form-group">
                  <label class="form-label">
                     {{ $t('word.definer') }}
                  </label>
                  <BaseSelect
                     v-model="localFunction.definer"
                     :options="[{value: '', name:$t('message.currentUser')}, ...workspace.users]"
                     :option-label="(user) => user.value === '' ? user.name : `${user.name}@${user.host}`"
                     :option-track-by="(user) => user.value === '' ? '' : `\`${user.name}\`@\`${user.host}\``"
                     class="form-select"
                  />
               </div>
            </div>
            <div class="column col-auto">
               <div class="form-group">
                  <label class="form-label">
                     {{ $t('word.returns') }}
                  </label>
                  <div class="input-group">
                     <BaseSelect
                        v-model="localFunction.returns"
                        class="form-select text-uppercase"
                        :options="[{ name: 'VOID' }, ...workspace.dataTypes]"
                        group-label="group"
                        group-values="types"
                        option-label="name"
                        option-track-by="name"
                        style="max-width: 150px;"
                     />
                     <input
                        v-if="customizations.parametersLength"
                        v-model="localFunction.returnsLength"
                        style="max-width: 150px;"
                        class="form-input"
                        type="number"
                        min="0"
                        :placeholder="$t('word.length')"
                     >
                  </div>
               </div>
            </div>
            <div v-if="customizations.comment" class="column">
               <div class="form-group">
                  <label class="form-label">
                     {{ $t('word.comment') }}
                  </label>
                  <input
                     v-model="localFunction.comment"
                     class="form-input"
                     type="text"
                  >
               </div>
            </div>
            <div class="column col-auto">
               <div class="form-group">
                  <label class="form-label">
                     {{ $t('message.sqlSecurity') }}
                  </label>
                  <BaseSelect
                     v-model="localFunction.security"
                     :options="['DEFINER', 'INVOKER']"
                     class="form-select"
                  />
               </div>
            </div>
            <div v-if="customizations.functionDataAccess" class="column col-auto">
               <div class="form-group">
                  <label class="form-label">
                     {{ $t('message.dataAccess') }}
                  </label>
                  <BaseSelect
                     v-model="localFunction.dataAccess"
                     :options="['CONTAINS SQL', 'NO SQL', 'READS SQL DATA', 'MODIFIES SQL DATA']"
                     class="form-select"
                  />
               </div>
            </div>
            <div v-if="customizations.functionDeterministic" class="column col-auto">
               <div class="form-group">
                  <label class="form-label d-invisible">.</label>
                  <label class="form-checkbox form-inline">
                     <input v-model="localFunction.deterministic" type="checkbox"><i class="form-icon" /> {{ $t('word.deterministic') }}
                  </label>
               </div>
            </div>
         </div>
      </div>
      <div class="workspace-query-results column col-12 mt-2 p-relative">
         <BaseLoader v-if="isLoading" />
         <label class="form-label ml-2">{{ $t('message.functionBody') }}</label>
         <QueryEditor
            v-show="isSelected"
            ref="queryEditor"
            v-model="localFunction.sql"
            :workspace="workspace"
            :schema="schema"
            :height="editorHeight"
         />
      </div>
      <WorkspaceTabPropsFunctionParamsModal
         v-if="isParamsModal"
         :local-parameters="localFunction.parameters"
         :workspace="workspace"
         :func="localFunction.name"
         @hide="hideParamsModal"
         @parameters-update="parametersUpdate"
      />
      <ModalAskParameters
         v-if="isAskingParameters"
         :local-routine="localFunction"
         :client="workspace.client"
         @confirm="runFunction"
         @close="hideAskParamsModal"
      />
   </div>
</template>

<script>
import { storeToRefs } from 'pinia';
import { useNotificationsStore } from '@/stores/notifications';
import { useWorkspacesStore } from '@/stores/workspaces';
import { uidGen } from 'common/libs/uidGen';
import BaseLoader from '@/components/BaseLoader';
import QueryEditor from '@/components/QueryEditor';
import WorkspaceTabPropsFunctionParamsModal from '@/components/WorkspaceTabPropsFunctionParamsModal';
import ModalAskParameters from '@/components/ModalAskParameters';
import Functions from '@/ipc-api/Functions';
import BaseSelect from '@/components/BaseSelect.vue';

export default {
   name: 'WorkspaceTabPropsFunction',
   components: {
      BaseLoader,
      QueryEditor,
      WorkspaceTabPropsFunctionParamsModal,
      ModalAskParameters,
      BaseSelect
   },
   props: {
      tabUid: String,
      connection: Object,
      function: String,
      isSelected: Boolean,
      schema: String
   },
   setup () {
      const { addNotification } = useNotificationsStore();
      const workspacesStore = useWorkspacesStore();

      const { getSelected: selectedWorkspace } = storeToRefs(workspacesStore);

      const {
         getWorkspace,
         refreshStructure,
         renameTabs,
         newTab,
         changeBreadcrumbs,
         setUnsavedChanges
      } = workspacesStore;

      return {
         addNotification,
         selectedWorkspace,
         getWorkspace,
         refreshStructure,
         renameTabs,
         newTab,
         changeBreadcrumbs,
         setUnsavedChanges
      };
   },
   data () {
      return {
         isLoading: false,
         isSaving: false,
         isParamsModal: false,
         isAskingParameters: false,
         originalFunction: null,
         localFunction: { sql: '' },
         lastFunction: null,
         sqlProxy: '',
         editorHeight: 300
      };
   },
   computed: {
      workspace () {
         return this.getWorkspace(this.connection.uid);
      },
      customizations () {
         return this.workspace.customizations;
      },
      isChanged () {
         return JSON.stringify(this.originalFunction) !== JSON.stringify(this.localFunction);
      },
      isDefinerInUsers () {
         return this.originalFunction
            ? this.workspace.users.some(user => this.originalFunction.definer === `\`${user.name}\`@\`${user.host}\``)
            : true;
      },
      isTableNameValid () {
         return this.localFunction.name !== '';
      },
      isInDataTypes () {
         let typeNames = [];
         for (const group of this.workspace.dataTypes) {
            typeNames = group.types.reduce((acc, curr) => {
               acc.push(curr.name);
               return acc;
            }, []);
         }
         return typeNames.includes(this.localFunction.returns);
      },
      schemaTables () {
         const schemaTables = this.workspace.structure
            .filter(schema => schema.name === this.schema)
            .map(schema => schema.tables);

         return schemaTables.length ? schemaTables[0].filter(table => table.type === 'table') : [];
      }
   },
   watch: {
      async schema () {
         if (this.isSelected) {
            await this.getFunctionData();
            this.$refs.queryEditor.editor.session.setValue(this.localFunction.sql);
            this.lastFunction = this.function;
         }
      },
      async function () {
         if (this.isSelected) {
            await this.getFunctionData();
            this.$refs.queryEditor.editor.session.setValue(this.localFunction.sql);
            this.lastFunction = this.function;
         }
      },
      async isSelected (val) {
         if (val) {
            this.changeBreadcrumbs({ schema: this.schema, function: this.function });

            setTimeout(() => {
               this.resizeQueryEditor();
            }, 200);

            if (this.lastFunction !== this.function)
               this.getRoutineData();
         }
      },
      isChanged (val) {
         this.setUnsavedChanges({ uid: this.connection.uid, tUid: this.tabUid, isChanged: val });
      }
   },
   async created () {
      await this.getFunctionData();
      this.$refs.queryEditor.editor.session.setValue(this.localFunction.sql);
      window.addEventListener('keydown', this.onKey);
   },
   mounted () {
      window.addEventListener('resize', this.resizeQueryEditor);
   },
   unmounted () {
      window.removeEventListener('resize', this.resizeQueryEditor);
   },
   beforeUnmount () {
      window.removeEventListener('keydown', this.onKey);
   },
   methods: {
      async getFunctionData () {
         if (!this.function) return;

         this.isLoading = true;
         this.localFunction = { sql: '' };
         this.lastFunction = this.function;

         const params = {
            uid: this.connection.uid,
            schema: this.schema,
            func: this.function
         };

         try {
            const { status, response } = await Functions.getFunctionInformations(params);
            if (status === 'success') {
               this.originalFunction = response;

               this.originalFunction.parameters = [...this.originalFunction.parameters.map(param => {
                  param._antares_id = uidGen();
                  return param;
               })];

               this.localFunction = JSON.parse(JSON.stringify(this.originalFunction));
               this.sqlProxy = this.localFunction.sql;
            }
            else
               this.addNotification({ status: 'error', message: response });
         }
         catch (err) {
            this.addNotification({ status: 'error', message: err.stack });
         }

         this.resizeQueryEditor();
         this.isLoading = false;
      },
      async saveChanges () {
         if (this.isSaving) return;
         this.isSaving = true;
         const params = {
            uid: this.connection.uid,
            func: {
               ...this.localFunction,
               schema: this.schema,
               oldName: this.originalFunction.name
            }
         };

         try {
            const { status, response } = await Functions.alterFunction(params);

            if (status === 'success') {
               const oldName = this.originalFunction.name;

               await this.refreshStructure(this.connection.uid);

               if (oldName !== this.localFunction.name) {
                  this.renameTabs({
                     uid: this.connection.uid,
                     schema: this.schema,
                     elementName: oldName,
                     elementNewName: this.localFunction.name,
                     elementType: 'function'
                  });

                  this.changeBreadcrumbs({ schema: this.schema, function: this.localFunction.name });
               }
               else
                  this.getFunctionData();
            }
            else
               this.addNotification({ status: 'error', message: response });
         }
         catch (err) {
            this.addNotification({ status: 'error', message: err.stack });
         }

         this.isSaving = false;
      },
      clearChanges () {
         this.localFunction = JSON.parse(JSON.stringify(this.originalFunction));
         this.$refs.queryEditor.editor.session.setValue(this.localFunction.sql);
      },
      resizeQueryEditor () {
         if (this.$refs.queryEditor) {
            const footer = document.getElementById('footer');
            const size = window.innerHeight - this.$refs.queryEditor.$el.getBoundingClientRect().top - footer.offsetHeight;
            this.editorHeight = size;
            this.$refs.queryEditor.editor.resize();
         }
      },
      optionsUpdate (options) {
         this.localFunction = options;
      },
      parametersUpdate (parameters) {
         this.localFunction = { ...this.localFunction, parameters };
      },
      runFunctionCheck () {
         if (this.localFunction.parameters.length)
            this.showAskParamsModal();
         else
            this.runFunction();
      },
      runFunction (params) {
         if (!params) params = [];

         let sql;
         switch (this.connection.client) { // TODO: move in a better place
            case 'maria':
            case 'mysql':
               sql = `SELECT \`${this.originalFunction.name}\` (${params.join(',')})`;
               break;
            case 'pg':
               sql = `SELECT ${this.originalFunction.name}(${params.join(',')})`;
               break;
            case 'mssql':
               sql = `SELECT ${this.originalFunction.name} ${params.join(',')}`;
               break;
            default:
               sql = `SELECT \`${this.originalFunction.name}\` (${params.join(',')})`;
         }

         this.newTab({ uid: this.connection.uid, content: sql, type: 'query', autorun: true });
      },
      showParamsModal () {
         this.isParamsModal = true;
      },
      hideParamsModal () {
         this.isParamsModal = false;
      },
      showAskParamsModal () {
         this.isAskingParameters = true;
      },
      hideAskParamsModal () {
         this.isAskingParameters = false;
      },
      onKey (e) {
         if (this.isSelected) {
            e.stopPropagation();
            if (e.ctrlKey && e.keyCode === 83) { // CTRL + S
               if (this.isChanged)
                  this.saveChanges();
            }
         }
      }
   }
};
</script>
