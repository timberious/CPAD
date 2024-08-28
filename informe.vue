<div id="header" data-sysreptor-generated="page-header">
    <div id="header-left">
      <img src="C:\Users\JhonSebastianSuarezH\"OneDrive - BITTIN SAS"\"J&S"\vscode\informes\logo_bittin.jpg" alt="logo" />
    </div>
    <div id="header-right">
      <span class="highlight">BITTIN SAS</span><br>
      Cra 7 #29 - 34<br>
      Bogotá, Colombia<br>
    </div>
  </div>
  
  
  <section id="page-cover">
  
    <img id="page-cover-logo" src='C:\Users\JhonSebastianSuarezH\"OneDrive - BITTIN SAS"\"J&S"\vscode\informes\logo_bittin.jpg' alt="">
  
    <div id="page-cover-infobox">
      <h1 id="page-cover-title">{{ report.title }}</h1>
    </div>
  
    <div id="page-cover-customer">
      <p>
        <strong>Customer:</strong><br>
        <strong><span class="highlight">{{ report.customer }}</span></strong><br>
        {{ report.report_date }}<br>
        v<template v-if="report.document_history.length > 0">
          {{ report.document_history[report.document_history.length - 1].version }}
        </template>
        <template v-else>0.0</template>
      </p>
    </div>
  
    <div id="page-cover-contact">
      <strong>Contact:</strong><br>
      {{ report.lead_pentester.name }}<br>
      {{ report.lead_pentester.phone }}<br>
      <strong><a :href="'mailto:' + report.lead_pentester.email" class="highlight">{{ report.lead_pentester.email }}</a></strong>
    </div>
  </section>
  
  <table-of-contents id="toc" v-slot="tocItems">
    <h1>Tabla De Contenidos</h1>
    <ul>
      <li v-for="item in tocItems" :class="'toc-level' + item.level">
        <ref :to="item.id" />
      </li>
    </ul>
    <pagebreak />
  </table-of-contents>
  
  
  <section>
    <h1 id="management-summary" class="in-toc">Resumen Ejecutivo</h1>
    <markdown :text="report.executive_summary" />
  </section>
  <pagebreak />
  
  <section>
    <h1 id="scope" class="in-toc">Metodología y objetivos</h1>
    <markdown :text="report.scope" />
  </section>
  <pagebreak />
  
  <section>
    <h1 id="findings-summary" class="in-toc">Consolidado de Vulnerabilidades</h1>
    <p>
      En la ejecución de pruebas de pentesting fueron identificadas las siguientes vulnerabilidades:
      <comma-and-join>
        <template #critical v-if="finding_stats.count_critical > 0"><strong class="risk-critical">{{ finding_stats.count_critical }} de nivel critico</strong></template>
        <template #high v-if="finding_stats.count_high > 0"><strong class="risk-high">{{ finding_stats.count_high }} de nivel alto</strong></template>
        <template #medium v-if="finding_stats.count_medium > 0"><strong class="risk-medium">{{ finding_stats.count_medium }} de nivel medio</strong></template>
        <template #low v-if="finding_stats.count_low > 0"><strong class="risk-low">{{ finding_stats.count_low }} de nivel bajo</strong></template>
        <template #info v-if="finding_stats.count_info > 0"><strong class="risk-info">{{ finding_stats.count_info }} de tipo informacional</strong></template>
      </comma-and-join>
      
    </p>
  
    <figure>
      <chart :width="15" :height="10" :config="{
          type: 'bar', 
          data: {
              labels: ['Critico', 'Alto', 'Medio', 'Bajo', 'Info'],
              datasets: [{
                  data: [
                      finding_stats.count_critical,
                      finding_stats.count_high,
                      finding_stats.count_medium,
                      finding_stats.count_low,
                      finding_stats.count_info
                  ],
                  backgroundColor: [
                      cssvar('--color-risk-critical'), 
                      cssvar('--color-risk-high'), 
                      cssvar('--color-risk-medium'), 
                      cssvar('--color-risk-low'), 
                      cssvar('--color-risk-info')
                  ],
              }]
          },
          options: {
              scales: {y: {beginAtZero: true, ticks: {precision: 0}}}, 
              plugins: {legend: {display: false}},
          }
      }" />
      <figcaption id="distribution-of-identified-vulnerabilities">Distribucion de vulnerabilidades identificadas</figcaption>
    </figure>
  
    <div>
      <p>En relación a lo anterior, en la siguiente tabla se puede observar el consolidado de las vulnerabilidades identificadas y su nivel de criticidad</p>
      <table class="finding-summary-table">
        <thead>
          <tr>
            <th>Vulnerabilidad</th>
            <th align="center">Criticidad</th>
            <th v-if="report.is_retest" align="center">Estado de remediación</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="finding in findings">
            <td>
              <ref :to="finding.id">{{ finding.title }}</ref>
            </td>
            <td align="center">
              <ref :to="finding.id" :class="'risk-' + finding.cvss.level">{{ lodash.capitalize(finding.cvss.level) }}</ref>
            </td>
            <td v-if="report.is_retest" align="center">
              <ref :to="finding.id" :class="'status-' + finding.retest_status.value">{{ finding.retest_status.label }}</ref>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  
    <div class="findings-list">
      <p>a continuación se listan las vulnerabilidades junto con una breve descripcion de cada hallazgo:</p>
      <div v-for="(finding, findingIndex) in findings">
        <h6 :id="finding.id + 'overview'">
          <ref :to="finding.id" class="finding-title">{{ finding.title }}</ref>
          (<span :class="'risk-' + finding.cvss.level"><a v-if="finding.cvss.vector.startsWith('CVSS:3.1')" :href="'https://www.first.org/cvss/calculator/3.1#' + finding.cvss.vector" class="link-none">{{ capitalize(finding.cvss.level) }}: {{ finding.cvss.score}}</a><a v-else-if="finding.cvss.vector.startsWith('CVSS:3.0')" :href="'https://www.first.org/cvss/calculator/3.0#' + finding.cvss.vector" class="link-none">{{ finding.cvss.score }}</a><template v-else>{{ capitalize(finding.cvss.level) }}: {{ finding.cvss.score }}</template></span><template v-if="report.is_retest || (finding.retest_status.value && finding.retest_status.value !== 'open')"> | <span :class="'status-' + (finding.retest_status.value || 'open')">{{ finding.retest_status.label || 'Offen' }}</span></template>)
        </h6>
  
        <div v-if="finding.affected_components && finding.affected_components.length > 0">
          Affects:
          <markdown v-if="finding.affected_components.length == 1" :text="finding.affected_components[0]" class="markdown-inline" />
          <ul v-else class="location-ul">
            <li v-for="component in finding.affected_components">
              <markdown :text="component" class="markdown-inline" />
            </li>
          </ul>
        </div>
        <markdown :text="finding.summary" />
      </div>
    </div>
  
  </section>
  <pagebreak />
  
  <section id="findings-list" class="findings-list">
    <h1 id="findings-details" class="in-toc">Detalle de vulnerabilidades</h1>
    <div v-for="(finding, findingIndex) in findings">
      <h2 :id="finding.id" class="in-toc" :data-toc-title="finding.title + ' (' + capitalize(finding.cvss.level) + ')'">
        <ref :to="finding.id + 'overview'" class="finding-title">{{ finding.title }}</ref>
      </h2>
  
      <div>
        <template v-if="report.is_retest || finding.retest_status.value !== 'open'">
          <strong>Estado de Remediación: </strong><span :class="'finding-status-' + finding.retest_status.value">{{ finding.retest_status.label }}</span><br>
        </template>
        <strong>Criticidad: </strong><span :class="'risk-' + finding.cvss.level">{{ capitalize(finding.cvss.level) }}</span><br>
        <strong>puntaje CVSS: </strong>
        <span :class="'risk-' + finding.cvss.level">
          <a v-if="finding.cvss.vector.startsWith('CVSS:3.1')" :href="'https://www.first.org/cvss/calculator/3.1#' + finding.cvss.vector" class="link-none">{{ finding.cvss.score}}</a>
          <a v-else-if="finding.cvss.vector.startsWith('CVSS:3.0')" :href="'https://www.first.org/cvss/calculator/3.0#' + finding.cvss.vector" class="link-none">{{ finding.cvss.score }}</a>
          <span v-else>{{ finding.cvss.score }}</span>
        </span><br>
        <template v-if="finding.affected_components && finding.affected_components.length > 0">
          <strong>Recurso Afectado: </strong>
          <markdown v-if="finding.affected_components.length == 1" :text="finding.affected_components[0]" class="markdown-inline" />
          <ul v-else class="location-ul">
            <li v-for="component in finding.affected_components">
              <markdown :text="component" class="markdown-inline" />
            </li>
          </ul>
        </template>
        <template v-if="finding.short_recommendation">
          <strong>Recomendaciones Inmediatas: </strong>
          <markdown :text="finding.short_recommendation" class="markdown-inline" /><br>
        </template>
      </div>
  
      <div>
        <h3>Resumen de Hallazgo</h3>
        <markdown :text="finding.summary" />
      </div>
  
      <div v-if="finding.retest_notes">
        <h3>Notas de estado de remediación</h3>
        <markdown :text="finding.retest_notes" />
      </div>
  
      <div>
        <h3 :id="finding.id + '-description'">Descripcion de vulnerabilidad</h3>
        <markdown :text="finding.description" />
      </div>
  
      <div>
        <h3 :id="finding.id + '-recommendation'">Recomendaciones</h3>
        <markdown :text="finding.recommendation" />
      </div>
  
      <div v-if="finding.references && finding.references.length > 0">
        <h3>Referencias</h3>
        <ul>
          <li v-for="reference in finding.references">
            <a :href="reference">{{ reference }}</a>
          </li>
        </ul>
      </div>
  
      <pagebreak />
    </div>
  </section>
  
  
  <section>
    <h1 id="document-history" class="in-toc">Historial de versiones</h1>
    <table>
      <thead>
        <tr>
          <th align="center">Version</th>
          <th align="center">Fecha</th>
          <th>Descripcion</th>
          <th>autor</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in report.document_history">
          <td align="center">{{ item.version }}</td>
          <td align="center">{{ item.date }}</td>
          <td>{{ item.description }}</td>
          <td>
            <comma-and-join>
              <template v-for="author in item.authors" #[author]>{{ author }}</template>
            </comma-and-join>
          </td>
        </tr>
      </tbody>
    </table>
  </section>
  
  <markdown>
    # Delcaracion de privacidad {#disclaimer .in-toc}
  
    En este apartado se realizaría la seccion de privacidad como "disclaimer", claramente
    esto se puede mover a la parte superior despues del indice y las tablas correspondientes
    
  </markdown>
  
  