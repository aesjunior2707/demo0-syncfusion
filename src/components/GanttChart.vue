<template>
  <div class="gantt-wrapper" :style="wrapperStyle">
    <div v-if="showLanguageSelector" id="language-selector-container" class="gantt-language-selector">
      <div class="language-selector">
        <label for="ganttLanguageSelector">🌐 Language:</label>
        <select id="ganttLanguageSelector" v-model="lang" @change="onLocaleChange">
          <option value="en-US">🇺🇸 English</option>
          <option value="pt-BR">🇧🇷 Português</option>
          <option value="es-ES">🇪🇸 Español</option>
        </select>
      </div>
    </div>
    <div ref="ganttEl" id="Gantt" class="gantt-host"></div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, watch, computed, nextTick } from 'vue'

const props = defineProps({
  modelValue: { type: Array, default: undefined },
  locale: { type: String, default: 'pt-BR' },
  showLanguageSelector: { type: Boolean, default: true },
  syncLocaleData: { type: Boolean, default: true },
  licenseKey: { type: String, default: '' },
  height: { type: [String, Number], default: '100%' },
  width: { type: [String, Number], default: '100%' }
})

const emit = defineEmits(['update:modelValue', 'ready', 'error', 'rowSelected', 'save', 'actionBegin', 'actionComplete'])

const ganttEl = ref(null)
const ganttInstance = ref(null)
const lang = ref(props.locale)

const wrapperStyle = computed(() => ({ height: typeof props.height === 'number' ? `${props.height}px` : props.height, width: typeof props.width === 'number' ? `${props.width}px` : props.width }))

// Utilities to inject external assets once
function ensureCss(id, href) {
  if (document.getElementById(id)) return
  const link = document.createElement('link')
  link.id = id
  link.rel = 'stylesheet'
  link.href = href
  document.head.appendChild(link)
}

function ensureScript(id, src) {
  return new Promise((resolve, reject) => {
    if (document.getElementById(id)) {
      if (window.ej) return resolve()
      return resolve()
    }
    const script = document.createElement('script')
    script.id = id
    script.src = src
    script.async = true
    script.onload = () => resolve()
    script.onerror = (e) => reject(e)
    document.head.appendChild(script)
  })
}

function checkDependencies() {
  return typeof window.ej !== 'undefined' && typeof getProjectDataByLocale === 'function'
}

// Localization payload (from localization.js)
function loadTranslations() {
  if (!(window.ej && window.ej.base && window.ej.base.L10n)) return
  window.ej.base.L10n.load({
    'en-US': {
      'gantt': {
        'id': 'ID', 'name': 'Task Name', 'startDate': 'Start Date', 'endDate': 'End Date', 'duration': 'Duration', 'progress': 'Progress', 'dependency': 'Predecessors', 'notes': 'Notes', 'baselineStartDate': 'Baseline Start Date', 'baselineEndDate': 'Baseline End Date', 'type': 'Type', 'offset': 'Offset', 'resourceName': 'Resources', 'resourceID': 'Resource ID', 'day': 'day', 'hour': 'hour', 'minute': 'minute', 'days': 'days', 'hours': 'hours', 'minutes': 'minutes',
        'addButton': 'Add', 'editButton': 'Edit', 'deleteButton': 'Delete', 'cancelButton': 'Cancel', 'saveButton': 'Save', 'updateButton': 'Update', 'expandAllButton': 'Expand All', 'collapseAllButton': 'Collapse All', 'searchButton': 'Search', 'excelExportButton': 'Excel Export', 'csvExportButton': 'CSV Export', 'pdfExportButton': 'PDF Export', 'zoomInButton': 'Zoom In', 'zoomOutButton': 'Zoom Out', 'zoomToFitButton': 'Zoom to Fit', 'prevTimeSpanButton': 'Previous Time Span', 'nextTimeSpanButton': 'Next Time Span', 'criticalPathButton': 'Critical Path',
        'addTask': 'Add Task', 'addSubTask': 'Add Sub Task', 'addMilestone': 'Add Milestone', 'editTask': 'Edit Task', 'deleteTask': 'Delete Task', 'save': 'Save', 'cancel': 'Cancel', 'taskInformation': 'Task Information', 'deleteConfirmation': 'Are you sure you want to delete Record?',
        'generalTab': 'General', 'customTab': 'Custom Fields', 'writeNotes': 'Write Notes', 'addDialogTitle': 'New Task', 'editDialogTitle': 'Task Information', 'update': 'Update', 'delete': 'Delete', 'search': 'Search',
        'filterMenuText': 'Filter', 'sortAscendingText': 'Sort Ascending', 'sortDescendingText': 'Sort Descending', 'clearFilter': 'Clear Filter',
        'taskBeforePredecessor_validation': 'The start date of task occurs before its predecessor task finish date.', 'taskAfterPredecessor_validation': 'The start date of task occurs after its predecessor task finish date.', 'taskSchedulingMode': 'Task Mode', 'changeSchedulingMode': 'Change Scheduling Mode', 'manual': 'Manual', 'auto': 'Auto', 'unscheduledTask': 'Unscheduled Task', 'taskmode': 'Task Mode',
        'weeks': 'Weeks', 'days': 'Days', 'hours': 'Hours', 'week': 'Week', 'day': 'Day', 'hour': 'Hour', 'month': 'Month', 'year': 'Year', 'months': 'Months', 'years': 'Years',
        'noRecordsText': 'No records to display', 'confirmDelete': 'Are you sure you want to delete this record?', 'from': 'From', 'to': 'To', 'taskType': 'Task Type', 'unassigned': 'Unassigned', 'okText': 'OK', 'confirm': 'Confirm', 'close': 'Close', 'taskName': 'Task Name', 'milestone': 'Milestone', 'expandAllTooltip': 'Expand All', 'collapseAllTooltip': 'Collapse All'
      }
    },
    'pt-BR': {
      'gantt': {
        'id': 'ID', 'name': 'Nome da Tarefa', 'startDate': 'Data de Início', 'endDate': 'Data Final', 'duration': 'Duração', 'progress': 'Progresso', 'dependency': 'Predecessores', 'notes': 'Notas', 'baselineStartDate': 'Data Base de Início', 'baselineEndDate': 'Data Base Final', 'type': 'Tipo', 'offset': 'Deslocamento', 'resourceName': 'Recursos', 'resourceID': 'ID do Recurso', 'day': 'dia', 'hour': 'hora', 'minute': 'minuto', 'days': 'dias', 'hours': 'horas', 'minutes': 'minutos',
        'addButton': 'Adicionar', 'editButton': 'Editar', 'deleteButton': 'Excluir', 'cancelButton': 'Cancelar', 'saveButton': 'Salvar', 'updateButton': 'Atualizar', 'expandAllButton': 'Expandir Tudo', 'collapseAllButton': 'Recolher Tudo', 'searchButton': 'Pesquisar', 'excelExportButton': 'Exportar Excel', 'csvExportButton': 'Exportar CSV', 'pdfExportButton': 'Exportar PDF', 'zoomInButton': 'Ampliar', 'zoomOutButton': 'Reduzir', 'zoomToFitButton': 'Ajustar ao Projeto', 'prevTimeSpanButton': 'Período Anterior', 'nextTimeSpanButton': 'Próximo Período', 'criticalPathButton': 'Caminho Crítico',
        'addTask': 'Adicionar Tarefa', 'addSubTask': 'Adicionar Subtarefa', 'addMilestone': 'Adicionar Marco', 'editTask': 'Editar Tarefa', 'deleteTask': 'Excluir Tarefa', 'save': 'Salvar', 'cancel': 'Cancelar', 'taskInformation': 'Informações da Tarefa', 'deleteConfirmation': 'Tem certeza que deseja excluir este registro?',
        'generalTab': 'Geral', 'customTab': 'Campos Personalizados', 'writeNotes': 'Escrever Notas', 'addDialogTitle': 'Nova Tarefa', 'editDialogTitle': 'Informações da Tarefa', 'update': 'Atualizar', 'delete': 'Excluir', 'search': 'Pesquisar',
        'filterMenuText': 'Filtrar', 'sortAscendingText': 'Ordenar Crescente', 'sortDescendingText': 'Ordenar Decrescente', 'clearFilter': 'Limpar Filtro',
        'taskBeforePredecessor_validation': 'A data de início da tarefa ocorre antes da data de término da tarefa predecessora.', 'taskAfterPredecessor_validation': 'A data de início da tarefa ocorre após a data de término da tarefa predecessora.', 'taskSchedulingMode': 'Modo da Tarefa', 'changeSchedulingMode': 'Alterar Modo de Agendamento', 'manual': 'Manual', 'auto': 'Automático', 'unscheduledTask': 'Tarefa Não Agendada', 'taskmode': 'Modo da Tarefa',
        'weeks': 'Semanas', 'days': 'Dias', 'hours': 'Horas', 'week': 'Semana', 'day': 'Dia', 'hour': 'Hora', 'month': 'Mês', 'year': 'Ano', 'months': 'Meses', 'years': 'Anos',
        'noRecordsText': 'Nenhum registro para exibir', 'confirmDelete': 'Tem certeza que deseja excluir este registro?', 'from': 'De', 'to': 'Para', 'taskType': 'Tipo de Tarefa', 'unassigned': 'Não Atribuído', 'okText': 'OK', 'confirm': 'Confirmar', 'close': 'Fechar', 'taskName': 'Nome da Tarefa', 'milestone': 'Marco', 'expandAllTooltip': 'Expandir Tudo', 'collapseAllTooltip': 'Recolher Tudo'
      }
    },
    'es-ES': {
      'gantt': {
        'id': 'ID', 'name': 'Nombre de Tarea', 'startDate': 'Fecha de Inicio', 'endDate': 'Fecha Final', 'duration': 'Duración', 'progress': 'Progreso', 'dependency': 'Predecesores', 'notes': 'Notas', 'baselineStartDate': 'Fecha Base de Inicio', 'baselineEndDate': 'Fecha Base Final', 'type': 'Tipo', 'offset': 'Desplazamiento', 'resourceName': 'Recursos', 'resourceID': 'ID del Recurso', 'day': 'día', 'hour': 'hora', 'minute': 'minuto', 'days': 'días', 'hours': 'horas', 'minutes': 'minutos',
        'addButton': 'Agregar', 'editButton': 'Editar', 'deleteButton': 'Eliminar', 'cancelButton': 'Cancelar', 'saveButton': 'Guardar', 'updateButton': 'Actualizar', 'expandAllButton': 'Expandir Todo', 'collapseAllButton': 'Contraer Todo', 'searchButton': 'Buscar', 'excelExportButton': 'Exportar Excel', 'csvExportButton': 'Exportar CSV', 'pdfExportButton': 'Exportar PDF', 'zoomInButton': 'Acercar', 'zoomOutButton': 'Alejar', 'zoomToFitButton': 'Ajustar al Proyecto', 'prevTimeSpanButton': 'Período Anterior', 'nextTimeSpanButton': 'Siguiente Período', 'criticalPathButton': 'Ruta Crítica',
        'addTask': 'Agregar Tarea', 'addSubTask': 'Agregar Subtarea', 'addMilestone': 'Agregar Hito', 'editTask': 'Editar Tarea', 'deleteTask': 'Eliminar Tarea', 'save': 'Guardar', 'cancel': 'Cancelar', 'taskInformation': 'Información de la Tarea', 'deleteConfirmation': '¿Está seguro de que desea eliminar este registro?',
        'generalTab': 'General', 'customTab': 'Campos Personalizados', 'writeNotes': 'Escribir Notas', 'addDialogTitle': 'Nueva Tarea', 'editDialogTitle': 'Información de la Tarea', 'update': 'Actualizar', 'delete': 'Eliminar', 'search': 'Buscar',
        'filterMenuText': 'Filtrar', 'sortAscendingText': 'Ordenar Ascendente', 'sortDescendingText': 'Ordenar Descendente', 'clearFilter': 'Limpiar Filtro',
        'taskBeforePredecessor_validation': 'La fecha de inicio de la tarea ocurre antes de la fecha de finalización de la tarea predecesora.', 'taskAfterPredecessor_validation': 'La fecha de inicio de la tarea ocurre después de la fecha de finalización de la tarea predecesora.', 'taskSchedulingMode': 'Modo de Tarea', 'changeSchedulingMode': 'Cambiar Modo de Programación', 'manual': 'Manual', 'auto': 'Automático', 'unscheduledTask': 'Tarea No Programada', 'taskmode': 'Modo de Tarea',
        'weeks': 'Semanas', 'days': 'Días', 'hours': 'Horas', 'week': 'Semana', 'day': 'Día', 'hour': 'Hora', 'month': 'Mes', 'year': 'Año', 'months': 'Meses', 'years': 'Años',
        'noRecordsText': 'No hay registros para mostrar', 'confirmDelete': '¿Está seguro de que desea eliminar este registro?', 'from': 'Desde', 'to': 'Hasta', 'taskType': 'Tipo de Tarea', 'unassigned': 'Sin Asignar', 'okText': 'OK', 'confirm': 'Confirmar', 'close': 'Cerrar', 'taskName': 'Nombre de Tarea', 'milestone': 'Hito', 'expandAllTooltip': 'Expandir Todo', 'collapseAllTooltip': 'Contraer Todo'
      }
    }
  })
}

// Demo datasets per locale (from localization.js)
const projectData = {
  'en-US': [
    { TaskID: 1, TaskName: 'Vehicle Planning and Design', StartDate: new Date('08/01/2025'), EndDate: new Date('08/05/2025'), subtasks: [
      { TaskID: 2, TaskName: 'Technical requirements analysis', StartDate: new Date('08/01/2025'), Duration: 2, Progress: 80 },
      { TaskID: 3, TaskName: 'Body design', StartDate: new Date('08/03/2025'), Duration: 2, Progress: 70, Predecessor: '2FS' },
      { TaskID: 4, TaskName: 'Component specification', StartDate: new Date('08/04/2025'), Duration: 1, Progress: 60, Predecessor: '3FS' },
    ] },
    { TaskID: 5, TaskName: 'Body Manufacturing', StartDate: new Date('08/06/2025'), EndDate: new Date('08/12/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 6, TaskName: 'Metal sheet cutting', StartDate: new Date('08/06/2025'), Duration: 2, Progress: 90 },
      { TaskID: 7, TaskName: 'Parts stamping', StartDate: new Date('08/08/2025'), Duration: 2, Progress: 75, Predecessor: '6FS' },
      { TaskID: 8, TaskName: 'Structure welding', StartDate: new Date('08/10/2025'), Duration: 3, Progress: 50, Predecessor: '7FS' },
    ] },
    { TaskID: 9, TaskName: 'Engine System', StartDate: new Date('08/13/2025'), EndDate: new Date('08/18/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 10, TaskName: 'Engine block machining', StartDate: new Date('08/13/2025'), Duration: 2, Progress: 60 },
      { TaskID: 11, TaskName: 'Engine assembly', StartDate: new Date('08/15/2025'), Duration: 2, Progress: 40, Predecessor: '10FS' },
      { TaskID: 12, TaskName: 'Transmission system', StartDate: new Date('08/17/2025'), Duration: 2, Progress: 30, Predecessor: '11FS' },
    ] },
    { TaskID: 13, TaskName: 'Chassis and Suspension', StartDate: new Date('08/19/2025'), EndDate: new Date('08/23/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 14, TaskName: 'Chassis welding', StartDate: new Date('08/19/2025'), Duration: 2, Progress: 45 },
      { TaskID: 15, TaskName: 'Brake system', StartDate: new Date('08/21/2025'), Duration: 2, Progress: 25, Predecessor: '14FS' },
      { TaskID: 16, TaskName: 'Suspension assembly', StartDate: new Date('08/22/2025'), Duration: 2, Progress: 20, Predecessor: '14FS' },
    ] },
    { TaskID: 17, TaskName: 'Final Assembly', StartDate: new Date('08/24/2025'), EndDate: new Date('08/29/2025'), Predecessor: '5FS;9FS;13FS', subtasks: [
      { TaskID: 18, TaskName: 'Engine installation on chassis', StartDate: new Date('08/24/2025'), Duration: 2, Progress: 15 },
      { TaskID: 19, TaskName: 'Body assembly', StartDate: new Date('08/26/2025'), Duration: 2, Progress: 10, Predecessor: '18FS' },
      { TaskID: 20, TaskName: 'Electrical and electronic system', StartDate: new Date('08/27/2025'), Duration: 2, Progress: 5, Predecessor: '19FS' },
    ] },
    { TaskID: 21, TaskName: 'Testing and Quality Control', StartDate: new Date('08/30/2025'), EndDate: new Date('08/31/2025'), Predecessor: '17FS', subtasks: [
      { TaskID: 22, TaskName: 'Functionality tests', StartDate: new Date('08/30/2025'), Duration: 1, Progress: 0 },
      { TaskID: 23, TaskName: 'Final quality inspection', StartDate: new Date('08/31/2025'), Duration: 1, Progress: 0, Predecessor: '22FS' },
    ] }
  ],
  'pt-BR': [
    { TaskID: 1, TaskName: 'Planejamento e Design do Veículo', StartDate: new Date('08/01/2025'), EndDate: new Date('08/05/2025'), subtasks: [
      { TaskID: 2, TaskName: 'Análise de requisitos técnicos', StartDate: new Date('08/01/2025'), Duration: 2, Progress: 80 },
      { TaskID: 3, TaskName: 'Design da carroceria', StartDate: new Date('08/03/2025'), Duration: 2, Progress: 70, Predecessor: '2FS' },
      { TaskID: 4, TaskName: 'Especificação de componentes', StartDate: new Date('08/04/2025'), Duration: 1, Progress: 60, Predecessor: '3FS' },
    ] },
    { TaskID: 5, TaskName: 'Fabricação da Carroceria', StartDate: new Date('08/06/2025'), EndDate: new Date('08/12/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 6, TaskName: 'Corte de chapas metálicas', StartDate: new Date('08/06/2025'), Duration: 2, Progress: 90 },
      { TaskID: 7, TaskName: 'Estampagem de peças', StartDate: new Date('08/08/2025'), Duration: 2, Progress: 75, Predecessor: '6FS' },
      { TaskID: 8, TaskName: 'Solda da estrutura', StartDate: new Date('08/10/2025'), Duration: 3, Progress: 50, Predecessor: '7FS' },
    ] },
    { TaskID: 9, TaskName: 'Sistema de Motorização', StartDate: new Date('08/13/2025'), EndDate: new Date('08/18/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 10, TaskName: 'Usinagem do bloco do motor', StartDate: new Date('08/13/2025'), Duration: 2, Progress: 60 },
      { TaskID: 11, TaskName: 'Montagem do motor', StartDate: new Date('08/15/2025'), Duration: 2, Progress: 40, Predecessor: '10FS' },
      { TaskID: 12, TaskName: 'Sistema de transmissão', StartDate: new Date('08/17/2025'), Duration: 2, Progress: 30, Predecessor: '11FS' },
    ] },
    { TaskID: 13, TaskName: 'Chassi e Suspensão', StartDate: new Date('08/19/2025'), EndDate: new Date('08/23/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 14, TaskName: 'Soldagem do chassi', StartDate: new Date('08/19/2025'), Duration: 2, Progress: 45 },
      { TaskID: 15, TaskName: 'Sistema de freios', StartDate: new Date('08/21/2025'), Duration: 2, Progress: 25, Predecessor: '14FS' },
      { TaskID: 16, TaskName: 'Montagem da suspensão', StartDate: new Date('08/22/2025'), Duration: 2, Progress: 20, Predecessor: '14FS' },
    ] },
    { TaskID: 17, TaskName: 'Montagem Final', StartDate: new Date('08/24/2025'), EndDate: new Date('08/29/2025'), Predecessor: '5FS;9FS;13FS', subtasks: [
      { TaskID: 18, TaskName: 'Instalação do motor no chassi', StartDate: new Date('08/24/2025'), Duration: 2, Progress: 15 },
      { TaskID: 19, TaskName: 'Montagem da carroceria', StartDate: new Date('08/26/2025'), Duration: 2, Progress: 10, Predecessor: '18FS' },
      { TaskID: 20, TaskName: 'Sistema elétrico e eletrônico', StartDate: new Date('08/27/2025'), Duration: 2, Progress: 5, Predecessor: '19FS' },
    ] },
    { TaskID: 21, TaskName: 'Testes e Controle de Qualidade', StartDate: new Date('08/30/2025'), EndDate: new Date('08/31/2025'), Predecessor: '17FS', subtasks: [
      { TaskID: 22, TaskName: 'Testes de funcionamento', StartDate: new Date('08/30/2025'), Duration: 1, Progress: 0 },
      { TaskID: 23, TaskName: 'Inspeção final de qualidade', StartDate: new Date('08/31/2025'), Duration: 1, Progress: 0, Predecessor: '22FS' },
    ] }
  ],
  'es-ES': [
    { TaskID: 1, TaskName: 'Planificación y Diseño del Vehículo', StartDate: new Date('08/01/2025'), EndDate: new Date('08/05/2025'), subtasks: [
      { TaskID: 2, TaskName: 'Análisis de requisitos técnicos', StartDate: new Date('08/01/2025'), Duration: 2, Progress: 80 },
      { TaskID: 3, TaskName: 'Diseño de la carrocería', StartDate: new Date('08/03/2025'), Duration: 2, Progress: 70, Predecessor: '2FS' },
      { TaskID: 4, TaskName: 'Especificación de componentes', StartDate: new Date('08/04/2025'), Duration: 1, Progress: 60, Predecessor: '3FS' },
    ] },
    { TaskID: 5, TaskName: 'Fabricación de la Carrocería', StartDate: new Date('08/06/2025'), EndDate: new Date('08/12/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 6, TaskName: 'Corte de chapas metálicas', StartDate: new Date('08/06/2025'), Duration: 2, Progress: 90 },
      { TaskID: 7, TaskName: 'Estampado de piezas', StartDate: new Date('08/08/2025'), Duration: 2, Progress: 75, Predecessor: '6FS' },
      { TaskID: 8, TaskName: 'Soldadura de la estructura', StartDate: new Date('08/10/2025'), Duration: 3, Progress: 50, Predecessor: '7FS' },
    ] },
    { TaskID: 9, TaskName: 'Sistema de Motorización', StartDate: new Date('08/13/2025'), EndDate: new Date('08/18/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 10, TaskName: 'Mecanizado del bloque del motor', StartDate: new Date('08/13/2025'), Duration: 2, Progress: 60 },
      { TaskID: 11, TaskName: 'Montaje del motor', StartDate: new Date('08/15/2025'), Duration: 2, Progress: 40, Predecesor: '10FS' },
      { TaskID: 12, TaskName: 'Sistema de transmisión', StartDate: new Date('08/17/2025'), Duration: 2, Progress: 30, Predecessor: '11FS' },
    ] },
    { TaskID: 13, TaskName: 'Chasis y Suspensión', StartDate: new Date('08/19/2025'), EndDate: new Date('08/23/2025'), Predecessor: '1FS', subtasks: [
      { TaskID: 14, TaskName: 'Soldadura del chasis', StartDate: new Date('08/19/2025'), Duration: 2, Progress: 45 },
      { TaskID: 15, TaskName: 'Sistema de frenos', StartDate: new Date('08/21/2025'), Duration: 2, Progress: 25, Predecessor: '14FS' },
      { TaskID: 16, TaskName: 'Montaje de la suspensión', StartDate: new Date('08/22/2025'), Duration: 2, Progress: 20, Predecessor: '14FS' },
    ] },
    { TaskID: 17, TaskName: 'Montaje Final', StartDate: new Date('08/24/2025'), EndDate: new Date('08/29/2025'), Predecessor: '5FS;9FS;13FS', subtasks: [
      { TaskID: 18, TaskName: 'Instalación del motor en el chasis', StartDate: new Date('08/24/2025'), Duration: 2, Progress: 15 },
      { TaskID: 19, TaskName: 'Montaje de la carrocería', StartDate: new Date('08/26/2025'), Duration: 2, Progress: 10, Predecessor: '18FS' },
      { TaskID: 20, TaskName: 'Sistema eléctrico y electrónico', StartDate: new Date('08/27/2025'), Duration: 2, Progress: 5, Predecessor: '19FS' },
    ] },
    { TaskID: 21, TaskName: 'Pruebas y Control de Calidad', StartDate: new Date('08/30/2025'), EndDate: new Date('08/31/2025'), Predecessor: '17FS', subtasks: [
      { TaskID: 22, TaskName: 'Pruebas de funcionamiento', StartDate: new Date('08/30/2025'), Duration: 1, Progress: 0 },
      { TaskID: 23, TaskName: 'Inspección final de calidad', StartDate: new Date('08/31/2025'), Duration: 1, Progress: 0, Predecessor: '22FS' },
    ] }
  ]
}

function getProjectDataByLocale(locale) {
  return projectData[locale] || projectData['en-US']
}

function displayPredecessors(field, data) {
  if (data.Predecessor) {
    return data.Predecessor.replace(/(\d+)FS/g, '$1').replace(/;/g, ', ')
  }
  return ''
}

function parsePredecessors(predecessorString) {
  if (!predecessorString || predecessorString.trim() === '') return ''
  const predecessorIds = predecessorString.split(',').map((id) => id.trim()).filter((id) => id !== '')
  const processed = predecessorIds.map((id) => {
    const numericId = id.replace(/[^\d]/g, '')
    if (numericId && !isNaN(numericId)) {
      if (/(\d+)(FS|SS|FF|SF)/.test(id)) return id
      return numericId + 'FS'
    }
    return null
  }).filter(Boolean)
  return processed.join(';')
}

function getAllTaskIds() {
  if (!ganttInstance.value) return []
  const taskIds = []
  function extractIds(data) {
    for (const item of data) {
      taskIds.push(item.TaskID)
      if (item.subtasks && item.subtasks.length > 0) extractIds(item.subtasks)
    }
  }
  const ds = ganttInstance.value.dataSource || []
  extractIds(ds)
  return taskIds
}

function validatePredecessors(predecessorString, currentTaskId) {
  if (!predecessorString || predecessorString.trim() === '') return { isValid: true, message: '' }
  const predecessorIds = predecessorString.split(',').map((id) => id.trim().replace(/[^\d]/g, ''))
  const allTaskIds = getAllTaskIds()
  for (let i = 0; i < predecessorIds.length; i++) {
    const predId = predecessorIds[i]
    if (predId === '') continue
    const numericPredId = parseInt(predId)
    if (!allTaskIds.includes(numericPredId)) {
      return { isValid: false, message: `Tarefa ${numericPredId} não existe.` }
    }
    if (numericPredId === currentTaskId) {
      return { isValid: false, message: 'Uma tarefa não pode ser predecessora de si mesma.' }
    }
  }
  return { isValid: true, message: '' }
}

function getNextTaskId() {
  let maxId = 0
  function findMaxId(data) {
    for (let i = 0; i < data.length; i++) {
      const item = data[i]
      if (item.TaskID > maxId) maxId = item.TaskID
      if (item.subtasks && item.subtasks.length > 0) findMaxId(item.subtasks)
    }
  }
  const ds = ganttInstance.value?.dataSource || []
  findMaxId(ds)
  return maxId + 1
}

function forceHideSpinner() {
  try {
    if (ganttInstance.value && ganttInstance.value.hideSpinner) ganttInstance.value.hideSpinner()
    const spinnerElements = document.querySelectorAll('.e-spinner-pane.e-spin-show')
    spinnerElements.forEach((spinner) => { spinner.classList.remove('e-spin-show'); spinner.style.display = 'none' })
  } catch {}
}

function updateColumnHeaders(locale) {
  if (!ganttInstance.value || !ganttInstance.value.columns) return
  const translations = {
    'en-US': { TaskID: 'ID', TaskName: 'Task Name', StartDate: 'Start Date', EndDate: 'End Date', Duration: 'Duration', Progress: 'Progress', Predecessor: 'Predecessors' },
    'pt-BR': { TaskID: 'ID', TaskName: 'Tarefa', StartDate: 'Início', EndDate: 'Fim', Duration: 'Duração', Progress: 'Prog.', Predecessor: 'Predecessores' },
    'es-ES': { TaskID: 'ID', TaskName: 'Tarea', StartDate: 'Inicio', EndDate: 'Fin', Duration: 'Duración', Progress: 'Prog.', Predecessor: 'Predecesores' }
  }
  const localeTexts = translations[locale] || translations['en-US']
  ganttInstance.value.columns.forEach((column) => { if (localeTexts[column.field]) column.headerText = localeTexts[column.field] })
}

function onLocaleChange() {
  if (!window.ej || !ganttInstance.value) return
  // Keep internal culture en-US for stability
  if (props.syncLocaleData && !props.modelValue) {
    ganttInstance.value.dataSource = getProjectDataByLocale(lang.value)
  }
  updateColumnHeaders(lang.value)
  ganttInstance.value.refresh()
}

function bindGlobalShortcuts() {
  function isTypingTarget(target) {
    const tag = (target && target.tagName ? target.tagName : '').toLowerCase()
    return tag === 'input' || tag === 'textarea' || (target && target.isContentEditable === true)
  }
  function keydownHandler(e) {
    const g = ganttInstance.value
    if (!g) return
    if (e.key === 'Escape') {
      const spinnerElements = document.querySelectorAll('.e-spinner-pane.e-spin-show')
      if (spinnerElements.length > 0) forceHideSpinner()
    }
    if (!e.ctrlKey || !e.shiftKey) {
      if (e.key === 'ArrowDown') {
        const selectedRowIndex = g.selectedRowIndex
        const totalRows = g.flatData ? g.flatData.length : 0
        const shouldCreateNewRow = (totalRows === 0) || (selectedRowIndex === totalRows - 1 && selectedRowIndex !== -1)
        if (shouldCreateNewRow) {
          e.preventDefault()
          const newRecord = { TaskID: getNextTaskId(), TaskName: '', StartDate: new Date(), Duration: 1, Progress: 0 }
          g.addRecord(newRecord, 'Below')
          setTimeout(() => {
            const newRowIndex = g.flatData.findIndex((row) => row.TaskID === newRecord.TaskID)
            if (newRowIndex !== -1) {
              g.selectRow(newRowIndex)
              g.treeGrid.editCell(newRowIndex, 'TaskName')
            }
          }, 150)
        }
      }
      return
    }
    if (isTypingTarget(e.target)) return
    if (e.key === 'ArrowRight' && typeof ganttInstance.value.indent === 'function') { e.preventDefault(); ganttInstance.value.indent() }
    else if (e.key === 'ArrowLeft' && typeof ganttInstance.value.outdent === 'function') { e.preventDefault(); ganttInstance.value.outdent() }
  }
  document.addEventListener('keydown', keydownHandler, false)
  return () => document.removeEventListener('keydown', keydownHandler, false)
}

function bindEnterToEdit() {
  const ganttElement = ganttEl.value
  if (!ganttElement) return () => {}
  let lastClickedRowIndex = -1
  function clickHandler(e) {
    const g = ganttInstance.value
    if (!g) return
    const treeGridElement = e.target.closest('.e-gantt-tree-grid')
    if (!treeGridElement) return
    const rowElement = e.target.closest('[role="row"]')
    if (rowElement && rowElement.closest('.e-content')) {
      const rowIndex = Array.from(rowElement.parentNode.children).indexOf(rowElement)
      if (rowIndex >= 0) { lastClickedRowIndex = rowIndex; setTimeout(() => { if (g.selectRow) g.selectRow(rowIndex) }, 10) }
    }
  }
  function keydownHandler(e) {
    const g = ganttInstance.value
    if (!g) return
    if (!e.target.closest('#Gantt')) return
    if (e.key === 'Enter' && !e.target.matches('input, textarea, [contenteditable]')) {
      try {
        let selectedRowIndex = g.selectedRowIndex
        if ((selectedRowIndex === undefined || selectedRowIndex === -1) && lastClickedRowIndex !== -1) selectedRowIndex = lastClickedRowIndex
        if (selectedRowIndex !== undefined && selectedRowIndex !== -1) {
          e.preventDefault()
          setTimeout(() => { if (g.treeGrid && g.treeGrid.editCell) g.treeGrid.editCell(selectedRowIndex, 'TaskName'); else if (g.startEdit) g.startEdit() }, 50)
        }
      } catch {}
    }
  }
  ganttElement.addEventListener('click', clickHandler, false)
  document.addEventListener('keydown', keydownHandler, false)
  return () => { ganttElement.removeEventListener('click', clickHandler, false); document.removeEventListener('keydown', keydownHandler, false) }
}

async function init() {
  try {
    ensureCss('ej2-theme-bootstrap5', 'https://cdn.syncfusion.com/ej2/30.2.7/bootstrap5.css')
    await ensureScript('ej2-bundle', 'https://cdn.syncfusion.com/ej2/30.2.7/dist/ej2.min.js')
    if (!(window.ej && window.ej.base && window.ej.base.setCulture)) throw new Error('Syncfusion EJ2 not available')

    // Setup culture and license
    try { window.ej.base.setCulture('en-US') } catch {}
    if (props.licenseKey && window.ej.base.registerLicense) {
      try { window.ej.base.registerLicense(props.licenseKey) } catch {}
    }

    loadTranslations()

    if (!checkDependencies()) {
      // Attach getProjectDataByLocale locally for checkDependencies()
      window.getProjectDataByLocale = getProjectDataByLocale
    }

    const ej = window.ej
    const options = {
      dataSource: props.modelValue ?? getProjectDataByLocale(lang.value),
      width: '100%',
      height: '100%',
      locale: 'en-US',
      dateFormat: 'dd/MM/yyyy',
      taskFields: { id: 'TaskID', name: 'TaskName', startDate: 'StartDate', endDate: 'EndDate', duration: 'Duration', progress: 'Progress', dependency: 'Predecessor', child: 'subtasks' },
      allowSorting: true,
      allowSelection: true,
      allowResizing: true,
      allowReordering: true,
      treeColumnIndex: 1,
      showColumnMenu: false,
      allowExcelExport: true,
      allowPdfExport: true,
      allowRowDragAndDrop: true,
      editSettings: { allowAdding: true, allowEditing: true, allowDeleting: true, allowTaskbarEditing: true, mode: 'Auto' },
      toolbar: ['Inserir', 'Delete', 'Clear', 'Cancel', 'ExpandAll', 'CollapseAll', 'Indent', 'Outdent', 'Search', 'ZoomIn', 'ZoomOut', 'ZoomToFit'],
      highlightWeekends: true,
      timelineSettings: { timelineUnitSize: 100, topTier: { unit: 'Month', format: 'MMM yyyy' }, bottomTier: { unit: 'Week', format: 'dd/MM' } },
      columns: [
        { field: 'TaskID', headerText: 'ID', width: 50, textAlign: 'Center', allowEditing: false },
        { field: 'TaskName', headerText: 'Tarefa', width: 250, allowEditing: true, clipMode: 'EllipsisWithTooltip' },
        { field: 'StartDate', headerText: 'Início', width: 90, allowEditing: true, editType: 'datepickeredit', edit: { params: { locale: 'en-US', format: 'M/d/yyyy' } } },
        { field: 'EndDate', headerText: 'Fim', width: 90, textAlign: 'Center', allowEditing: true, editType: 'datepickeredit', edit: { params: { locale: 'en-US', format: 'M/d/yyyy' } } },
        { field: 'Duration', headerText: 'Duração', width: 80, textAlign: 'Center', allowEditing: true, editType: 'numericedit', edit: { params: { min: 1, max: 999, step: 1, format: 'n0' } } },
        { field: 'Progress', headerText: 'Prog.', width: 70, textAlign: 'Center', allowEditing: true },
        { field: 'Predecessor', headerText: 'Predecessores', width: 120, textAlign: 'Left', allowEditing: true, clipMode: 'EllipsisWithTooltip', valueAccessor: displayPredecessors }
      ],
      toolbarClick: (args) => {
        if (args && args.item && args.item.id === 'Gantt_Clear') {
          ganttInstance.value.dataSource = []
          ganttInstance.value.refresh()
        }
        if (args && args.item && args.item.id === 'Gantt_Inserir') {
          const newRecord = { TaskID: getNextTaskId(), TaskName: '', StartDate: new Date(), Duration: 1, Progress: 0 }
          ganttInstance.value.addRecord(newRecord, 'Below')
          setTimeout(() => {
            const newRowIndex = ganttInstance.value.flatData.findIndex((row) => row.TaskID === newRecord.TaskID)
            if (newRowIndex !== -1) { ganttInstance.value.selectRow(newRowIndex); ganttInstance.value.treeGrid.editCell(newRowIndex, 'TaskName') }
          }, 150)
        }
      },
      labelSettings: { leftLabel: 'TaskName', rightLabel: 'Progress', taskLabel: '${Progress}%' },
      splitterSettings: { columnIndex: 3 },
      projectStartDate: new Date('08/01/2025'),
      projectEndDate: new Date('08/31/2025'),
      zoomSettings: { enable: true, zoomIn: true, zoomOut: true, zoomToFit: true },
      rowDrop: (args) => {
        try {
          setTimeout(() => { if (ganttInstance.value && ganttInstance.value.hideSpinner) ganttInstance.value.hideSpinner() }, 100)
        } catch {
          if (ganttInstance.value && ganttInstance.value.hideSpinner) ganttInstance.value.hideSpinner()
          if (args) args.cancel = true
        }
      },
      actionBegin: (args) => {
        emit('actionBegin', args)
        if (args.requestType === 'rowDropping') { /* noop logging removed */ }
        if (args.requestType === 'save' && args.data && args.data.Predecessor !== undefined) {
          const originalValue = args.data.Predecessor
          const validation = validatePredecessors(originalValue, args.data.TaskID)
          if (!validation.isValid) { args.cancel = true; alert('Erro nos predecessores: ' + validation.message); return }
          const processed = parsePredecessors(originalValue)
          args.data.Predecessor = processed
        }
        if (args.requestType === 'validateLinkedTask') { args.validateMode = { respectLink: true } }
      },
      actionComplete: (args) => {
        emit('actionComplete', args)
        if (args.requestType === 'rowDropping') {
          setTimeout(() => { if (ganttInstance.value && ganttInstance.value.hideSpinner) ganttInstance.value.hideSpinner() }, 50)
        }
        if (args.requestType === 'save' && args.data) emit('save', args.data)
        // Sync v-model if using external data
        if (props.modelValue) emit('update:modelValue', ganttInstance.value.dataSource)
      },
      rowSelected: (args) => emit('rowSelected', args),
      keyPressed: (args) => {
        if (args.key === 'Enter' && !args.target.classList.contains('e-input')) {
          const selectedRowIndex = ganttInstance.value.selectedRowIndex
          if (selectedRowIndex !== undefined && selectedRowIndex !== -1) {
            args.preventDefault()
            setTimeout(() => { if (ganttInstance.value.treeGrid && ganttInstance.value.treeGrid.editCell) ganttInstance.value.treeGrid.editCell(selectedRowIndex, 'TaskName') }, 50)
          }
        }
      }
    }

    ganttInstance.value = new ej.gantt.Gantt(options)
    ganttInstance.value.appendTo(ganttEl.value)

    // Fit to project after first bind
    ganttInstance.value.dataBound = function () {
      if (!ganttInstance.value.isInitialLoad) {
        ganttInstance.value.isInitialLoad = true
        setTimeout(() => { if (ganttInstance.value && ganttInstance.value.fitToProject) ganttInstance.value.fitToProject() }, 100)
      }
    }

    // Spinner watchdog
    const spinnerInterval = setInterval(() => {
      const spinners = document.querySelectorAll('.e-spinner-pane.e-spin-show')
      if (spinners.length > 0) forceHideSpinner()
    }, 5000)

    const unbindShortcuts = bindGlobalShortcuts()
    const unbindEnterEdit = bindEnterToEdit()

    onBeforeUnmount(() => {
      clearInterval(spinnerInterval)
      if (unbindShortcuts) unbindShortcuts()
      if (unbindEnterEdit) unbindEnterEdit()
      try { ganttInstance.value?.destroy() } catch {}
    })

    emit('ready', ganttInstance.value)
  } catch (e) {
    emit('error', e)
  }
}

onMounted(async () => {
  await nextTick()
  await init()
})

watch(() => props.locale, (val) => { lang.value = val; onLocaleChange() })
watch(() => props.modelValue, (val) => {
  if (ganttInstance.value && Array.isArray(val)) { ganttInstance.value.dataSource = val; ganttInstance.value.refresh() }
})

// Expose a minimal control API
function addTaskBelow() {
  if (!ganttInstance.value) return
  const newRecord = { TaskID: getNextTaskId(), TaskName: '', StartDate: new Date(), Duration: 1, Progress: 0 }
  ganttInstance.value.addRecord(newRecord, 'Below')
}
function clearAll() { if (ganttInstance.value) { ganttInstance.value.dataSource = []; ganttInstance.value.refresh() } }
function expandAll() { ganttInstance.value?.expandAll() }
function collapseAll() { ganttInstance.value?.collapseAll() }
function zoomIn() { ganttInstance.value?.zoomIn() }
function zoomOut() { ganttInstance.value?.zoomOut() }
function zoomToFit() { ganttInstance.value?.fitToProject() }
function getInstance() { return ganttInstance.value }

defineExpose({ addTaskBelow, clearAll, expandAll, collapseAll, zoomIn, zoomOut, zoomToFit, getInstance })
</script>

<style scoped>
.gantt-wrapper { height: 100%; width: 100%; }
.gantt-host { height: 100%; width: 100%; }

/* LANGUAGE SELECTOR STYLING (preservado do projeto original) */
.gantt-language-selector {
  position: fixed;
  top: 10px;
  right: 10px;
  z-index: 1000;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 6px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  padding: 8px 12px;
  border: 1px solid #e5e7eb;
}
.language-selector { display: flex; align-items: center; gap: 8px; }
.language-selector label { font-size: 11px; font-weight: 500; color: #374151; white-space: nowrap; }
.language-selector select { font-size: 11px; padding: 4px 8px; border: 1px solid #e5e7eb; border-radius: 4px; background: #ffffff; color: #374151; cursor: pointer; min-width: 120px; outline: none; }
.language-selector select:hover { border-color: #007acc; }
.language-selector select:focus { border-color: #007acc; box-shadow: 0 0 0 2px rgba(0, 122, 204, 0.1); }
</style>
