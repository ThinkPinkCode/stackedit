<template>
  <modal-inner aria-label="Export All Files">
    <div class="modal__content">
      <p>Please choose a file format for your exports.</p>

      <form-entry>
        <select class="textfield" slot="field" v-model="selectedTemplate" @keydown.enter="resolve()">
          <optgroup label="Markdown">Markdown
          <option value="Markdown">Markdown</option>      //TODO: find better way to display
          </optgroup>
          <optgroup label="HTML">
          <option v-for="(template, id) in allTemplates" :key="id" :value="id">
            {{ template.name }}
          </option>
          </optgroup>
        </select>
        <div class="form-entry__actions">
          <a href="javascript:void(0)" @click="configureTemplates">Configure templates</a>
        </div>
      </form-entry>
    </div>
    <div class="modal__button-bar">
      <button class="button" @click="config.reject()">Cancel</button>
      <button class="button" @click="resolve()">Ok</button>
    </div>
  </modal-inner>
</template>

<script>
  import exportSvc from '../../services/exportSvc';
  import modalTemplate from './common/modalTemplate';

  export default modalTemplate({
    data: () => ({
      result: '',
    }),
    computedLocalSettings: {
      selectedTemplate: 'htmlExportTemplate',
    },
    mounted() {
      let timeoutId;
      this.$watch('selectedTemplate', (selectedTemplate) => {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(async () => {
          const currentFile = this.$store.getters['file/current'];
          const html = await exportSvc.applyTemplate(
            currentFile.id,
            this.allTemplates[selectedTemplate],
          );
          this.result = html;
        }, 10);
      }, {
        immediate: true,
      });
    },
    methods: {
      resolve() {
        const allFiles = this.$store.getters['file/items'];
        if (this.selectedTemplate === 'Markdown') {
          exportSvc.exportAllToDisk(allFiles, 'md');
        } else {
          const { config } = this;
          config.resolve();
          exportSvc.exportAllToDisk(allFiles, 'html', this.allTemplates[this.selectedTemplate]);
        }
      },
    },
  });
</script>
