
<style lang="sass">
@import ../../../assets/sass/variables




#files-form
  padding-left: 40px
  padding-right: 20px

    
  #invalid-build-alert,
  #invalid-file-combo-alert  
    margin-left: 40px
    padding: 5px
    min-width: 510px

  .input-group.radio
    margin-top: 0px
    margin-bottom: 0px

  .v-radio
    padding-right: 0px
    margin-bottom: 0px

  .radio label,
  .v-radio label
    line-height: 25px
    padding-top: 3px
    font-size: 13px
    margin: 0px

  .input-group.radio-group
    padding-top: 0px

  .v-input--radio-group
    padding-top: 0px
    margin-top: 0px

  .input-group__selections__comma, .v-select__selection--comma
    font-size: 13px

  .input-group.input-group--selection-controls.switch,
  .v-input--selection-controls.v-input--switch,
    label
      font-weight: normal
      font-size: 13px
      padding-left: 5px

  .radio-group
    padding-top: 0px
    .input-group__input
      min-height: 25px

  .loader
    display: inline-block
    margin-right: 2px

    img
      width: 20px
      height: 20px

  .sample-label
    span
      margin-top: 2px
      margin-bottom: 2px
      vertical-align: top
      margin-left: 0px
      font-size: 15px
      color: $app-color
      display: inline-block
      margin-right: 20px
    .switch
      display: inline-block
      width: 100px

  #info-alert
    padding: 8px !important
  #info-message
    font-size: 13px
    color: $text-color

    pre
      display: inline-block
      vertical-align: top
      padding-top: 0px
      padding-bottom: 0px
      font-size: 13px
      color: black
      margin-bottom: 0px


  button

    height: 24px !important;
    min-width: 100px;

    &.action-button

      height: 30px !important
      min-width: 77px
      color: $app-button-color

      &.cancel-button
        color: $text-color

    &.load-button
      padding: 0px
      height: 30px !important
      background-color: $app-button-color !important
      color: white !important
      min-width: 100px !important
      margin: 0px

      &.disabled
        opacity: 0.20 !important

      &.v-btn--disabled
        opacity: 0.20 !important


  .v-text-field__slot,
  .v-select__slot
    input
      font-size: 12px
      color: $text-color

  .sample-label
    .v-input--switch
      margin-bottom: 10px
      margin-top: 0px

  .siblings-disabled
      font-size: 13px !important
      color: $text-color !important
      padding-top: 2px !important
</style>

<template>
   <v-dialog v-model="showFilesDialog" persistent max-width="890" >
      <v-card class="full-width" style="min-height:0px;max-height:670px;overflow-y:scroll">
          <v-form id="files-form">

            <v-dialog width="500" v-model="areAnyDuplicates" v-if="warningOpen">
              <v-card class="info-card full-width" id="remove-filter-card">
                <v-card-title style="justify-content:space-between">
                  <span class="info-title">{{errorTitle}}</span>
                </v-card-title>
                             

                <v-card-text class="remove-filter-description" style="overflow-wrap: break-word">
                  <div v-for="msg in errorMsgArray">
                    {{msg}}
                  </div>
                </v-card-text>
                <v-btn @click="loadReady = false; warningOpen = false;" color="normal">Cancel</v-btn>
                <v-btn @click="loadReady = true; warningOpen = false;" color="error">Continue</v-btn>
              </v-card>
            </v-dialog>

            <v-layout row nowrap class="mt-0">
             <v-card-title class="headline">Files</v-card-title>
               <div style="display:flex; flex-direction:column">
                <v-alert id="invalid-file-combo-alert" v-if="!isValidFileCombo"  :value="true" color="error" icon="warning" outline>
                    Invalid file combination for trio. If one of the trio samples loads alignments but not a variant file, the other samples must follow the same pattern, loading the alignment files only.
                  </v-alert>

                <v-alert id="invalid-build-alert" v-if="!isValidBuild"  :value="true" color="error" icon="warning" outline>
                    {{ invalidBuildMessage}}
                </v-alert>
              </div>
              <v-flex xs12 class="mt-2 text-xs-right">
                <div class="loader" v-show="inProgress">
                  <img src="../../../assets/images/wheel.gif">
                </div>

                <v-btn class="load-button action-button"
                  @click="onLoad"
                  :disabled="!isValid || !buildName || !isValidFileCombo">
                  Load
                </v-btn>

                <v-btn class="cancer-button action-button" @click="onClose">
                 Close
               </v-btn>
              </v-flex>
            </v-layout>


            <v-layout row nowrap class="mt-0">
              <v-flex xs12 class="mt-2 mb-2 text-xs-left" v-if="infoMessage && infoMessage.length > 0">
                <v-alert id="info-alert"
                  :value="true"
                  color="info"
                  icon="info"
                  outline
                >
                  <div id="info-message" v-html="infoMessage">
                  </div>
                </v-alert>
              </v-flex>
            </v-layout>

            <v-layout row nowrap class="mt-0">
              <v-flex class="mt-0" style="max-width: 90px;margin-right: 10px;" >
                  <v-radio-group v-model="mode" @change="onModeChanged"  hide-details column>
                        <v-radio label="Single"  value="single"></v-radio>
                        <v-radio label="Trio"    value="trio"></v-radio>
                  </v-radio-group>
              </v-flex>

              <v-flex style="width:90px;margin-right:10px" class="mt-2" >
                  <v-switch  label="Separate URL for index" hide-details v-model="separateUrlForIndex">
                  </v-switch>
              </v-flex>


              <v-flex style="max-width:160px" >
                <v-select
                  label="Species"
                  hide-details
                  v-model="speciesName"
                  :items="speciesList"
                ></v-select>
              </v-flex>

              <v-flex style="max-width:160px" class="ml-2">
                <v-select
                  label="Genome Build"
                  hide-details
                  v-model="buildName"
                  :items="buildList"
                  :class="clazzAttention"
                ></v-select>
               </v-flex>

              <v-flex style="max-width:160px" class="ml-2">
                  <v-select
                    :items="demoActions"
                    item-value="value"
                    item-text="display"
                    @input="onLoadDemoData"
                    v-model="demoAction"
                    hide-details
                    label="Demo data"></v-select>
              </v-flex>

            </v-layout>
            <v-layout row wrap class="mt-3">



               <v-flex xs12
                 v-for="rel in rels[mode]"
                    :key="rel"
                    :id="rel"
                    v-if="modelInfoMap && modelInfoMap[rel] && Object.keys(modelInfoMap[rel]).length > 0">

                  <sample-data
                   ref="sampleDataRef"
                   v-if="modelInfoMap && modelInfoMap[rel] && Object.keys(modelInfoMap[rel]).length > 0"
                   :modelInfo="modelInfoMap[rel]"
                   :separateUrlForIndex="separateUrlForIndex"
                   @sample-data-changed="validate"
                   @samples-available="onSamplesAvailable"
                   @sample-error="onSampleError"
                  >
                </sample-data>
               </v-flex>

            </v-layout >

            <v-layout row no-wrap class="pr-2 pb-4 pt-3">

               <v-flex  class="sample-label mt-3 pl-2 pr-2" >
                <span
                 dark small >
                  siblings
                </span>
                <span class="siblings-disabled" v-if="!(possibleSibs && possibleSibs.length > 0)" >
                 **Proband vcf must contain sibling data to enable sibling selection
               </span>
               </v-flex>
                <v-flex  class=" pl-2 pr-3" >
                 <v-autocomplete
                  v-bind:disabled="!(possibleSibs && possibleSibs.length > 0)"
                  label="Affected Siblings"
                  multiple
                  v-model="affectedSibs"
                  :items="possibleSibs"
                  item-text="sample"
                  item-value="sample"
                  hide-details
                  >
                </v-autocomplete>
               </v-flex>

               <v-flex   class="pr-2">
                 <v-autocomplete
                  v-bind:disabled="!(possibleSibs && possibleSibs.length > 0)"
                  label="Unaffected Siblings"
                  multiple
                  v-model="unaffectedSibs"
                  item-text="sample"
                  item-value="sample"
                  :items="possibleSibs"
                  hide-details
                  >
                </v-autocomplete>
               </v-flex>
            </v-layout>

          </v-form>
      </v-card>
  </v-dialog>
</template>

<script>

import SampleData          from '../partials/SampleData.vue'


export default {
  name: 'files-dialog',
  components: {
    SampleData
  },
  props: {
    cohortModel: null,
    showDialog: null,
    showDialogForAnalysis: null,
    launchedFromDemo: null,
    infoMessage: null
  },
  data() {
    return {
      showFilesDialog: false,
      isValid: false,
      isValidFileCombo: true,
      isValidBuild: true,
      invalidBuildMessage: '',
      areAnyDuplicates: false,
      loadReady: true,
      warningOpen: false,
      errorMsgArray: [],
      errorTitle: "",
      mode: 'single',
      speciesList: [],
      speciesName: null,
      buildName: null,
      activeTab: null,
      modelInfoMap: {
        proband: {},
        mother: {},
        father: {}
      },
      rels: {
        single: ['proband'],
        trio: ['proband', 'mother', 'father']
      },
      demoActions: [
        {'display': 'Demo WES trio', 'value': 'exome'},
        {'display': 'Demo WGS trio', 'value': 'genome'}
      ],
      demoAction: null,
      separateUrlForIndex: false,
      possibleSibs: null,
      affectedSibs: null,
      unaffectedSibs: null,
      inProgress: false,
      isDemo: false
    }
  },
  watch: {
    loadReady: function(){
      let self = this;
      if(self.loadReady) {
        self.cohortModel.promiseAddClinvarSample()
        .then(function () {
          let affectedSibList = self.possibleSibs.filter(function(possibleSib) {
            return self.affectedSibs != null && self.affectedSibs.indexOf(possibleSib.sample) >= 0;
          })
          let unaffectedSibList = self.possibleSibs.filter(function(possibleSib) {
            return self.unaffectedSibs !=null && self.unaffectedSibs.indexOf(possibleSib.sample) >= 0;
          })
          return self.cohortModel.promiseSetSibs(affectedSibList, unaffectedSibList)
        })
        .then(function () {
          self.cohortModel.setAffectedInfo(true);
          self.cohortModel.isLoaded = true;
          self.cohortModel.getCanonicalModels().forEach(function (model) {
            if (model.name == null || model.name.length == 0) {
              model.name = model.relationship;
            }
          });
          self.cohortModel.sortSampleModels();
        })
        .then(function () {
          let performAnalyzeAll = false;
          if (self.demoAction) {
            if (!self.showDialogForAnalysis) {
              performAnalyzeAll = true;
            }
          }
          let clearSession = !self.showDialogForAnalysis;

          self.inProgress = false;
          self.$emit("on-files-loaded", performAnalyzeAll, clearSession);
          self.showFilesDialog = false;
        });
      }
    },
    showDialog: function() {
      if (this.cohortModel && this.showDialog) {
        this.showFilesDialog = true
        if(this.mode !== 'trio') {
          this.mode = this.cohortModel.mode;
        }
        this.init();
      }
    },
    showDialogForAnalysis: function() {
      if (this.cohortModel && this.showDialogForAnalysis) {
        this.showFilesDialog = true
        if(this.mode !== 'trio') {
          this.mode = this.cohortModel.mode;
        }
        this.init();
      }
    },
    showFilesDialog: function() {
      if (!this.showFilesDialog) {
        this.$emit("on-cancel");
      }
    },
    launchedFromDemo: function() {
      if(this.launchedFromDemo) {
        this.buildName = this.cohortModel.genomeBuildHelper.getCurrentBuildName();
      }
    },
    buildName: function() {
      this._validateBuild()
    }
  },
  methods: {
    checkValidExtensions: function(sms){
      let self = this;
      for(let i = 0; i < sms.length; i++){
        if(sms[i].bam || sms[i].vcf) {
          let bamUrl = sms[i].bam ? sms[i].bam.bamUri : null;
          let baiUrl = sms[i].bam ? sms[i].bam.baiUri : null;
          let vcfUrl = sms[i].vcf ? sms[i].vcf.vcfURL : null;
          let tbiUrl = sms[i].vcf ? sms[i].vcf.tbiUrl : null;

          if (!vcfUrl) {
            vcfUrl = sms[i].vcf ? sms[i].vcf.getVcfFile() : null;
            tbiUrl = sms[i].vcf ? sms[i].vcf.getTbiURL() : null;
          }

          if (bamUrl
              && bamUrl.split('.').pop() !== "bam"
              && bamUrl.split('.').pop() !== "cram") {
            self.errorTitle = "Bam file extension warning";
            let errorMsg = "The bam file path does not end with a .bam extension " + bamUrl;
            self.errorMsgArray.push(errorMsg);
            self.warningOpen = true;
            self.areAnyDuplicates = true;
            self.loadReady = false;
          }
          if (baiUrl
            && baiUrl.split('.').pop() !== "bai"
            && baiUrl.split('.').pop() !== "crai") {
            self.errorTitle = "Bam index file extension warning";
            let errorMsg = "The bam index file path does not end with a .bai extension " + baiUrl;
            self.errorMsgArray.push(errorMsg);
            self.warningOpen = true;
            self.areAnyDuplicates = true;
            self.loadReady = false;
          }
          if (vcfUrl && vcfUrl.split('.').pop() !== "gz") {
            self.errorTitle = "Vcf file extension warning";
            let errorMsg = "The vcf index file path does not end with a .vcf.gz extension " + vcfUrl;
            self.errorMsgArray.push(errorMsg);
            self.warningOpen = true;
            self.areAnyDuplicates = true;
            self.loadReady = false;
          }
          if (tbiUrl && tbiUrl.split('.').pop() !== "tbi" && tbiUrl && tbiUrl.split('.').pop() !== "csi" ) {
            self.errorTitle = "Vcf index file extension warning";
            let errorMsg = "The vcf index file path does not end with a .tbi extension " + tbiUrl;
            self.errorMsgArray.push(errorMsg);
            self.warningOpen = true;
            self.areAnyDuplicates = true;
            self.loadReady = false;
          }
        }
      }
    },
    checkForDuplicates: function(sms){
      let self = this;
      let names = sms.map(function(obj) {
        return obj.name;
      }).concat(self.affectedSibs, self.unaffectedSibs);

      names.forEach(function (element, index, arr) {
        if (element && arr.indexOf(element) !== index) {
          self.errorTitle = "Duplicate Ids";
          let errorMsg = "Duplicate ids detected for " + element;
          self.errorMsgArray.push(errorMsg);
          self.warningOpen = true;
          self.areAnyDuplicates = true;
          self.loadReady = false;
        }
      });

      if(sms[0].bam) {
        sms.map(function (obj) {
          return obj.bam.bamUri;
        }).forEach(function (element, index, arr) {
          if (arr.indexOf(element) !== index) {
            self.errorTitle = "Duplicate Bam Files";
            let errorMsg = "Duplicate Bam detected for file: " + element;
            self.errorMsgArray.push(errorMsg);
            self.warningOpen = true;
            self.areAnyDuplicates = true;
            self.loadReady = false;
          }
        });
      }
    },

    onLoad: function() {
      let self = this;
      if(self.isDemo) {
        self.$ga.event('data_type', 'Demo Data', 'Files demo dataset');
      }
      else {
        self.$ga.event('data_type', 'Custom Data', 'Custom dataset');
      }
      self.cohortModel.setMode(self.mode);
      self.cohortModel.genomeBuildHelper.setCurrentBuild(self.buildName);
      self.cohortModel.genomeBuildHelper.setCurrentSpecies(self.speciesName);

      let sms = self.cohortModel.sampleModels;
      self.areAnyDuplicates = false;
      self.loadReady = true;
      self.errorMsgArray = [];
      self.checkForDuplicates(sms);
      self.checkValidExtensions(sms);

      if(self.errorMsgArray.length > 1){
        self.errorTitle = "Multiple warnings";
      }

      if(self.loadReady) {
        self.inProgress = true;
        self.cohortModel.promiseAddClinvarSample()
        .then(function () {
          return self.setSibsInCohortModel();
        })
        .then(function () {
          self.cohortModel.setAffectedInfo(true);
          self.cohortModel.isLoaded = true;
          self.cohortModel.getCanonicalModels().forEach(function (model) {
            if (model.name == null || model.name.length == 0) {
              model.name = model.relationship;
            }
          });
          self.cohortModel.sortSampleModels();
        })
        .then(function () {
          let performAnalyzeAll = self.demoAction ? true : false;
          self.inProgress = false;
          let clearCache = true;
          self.$emit("on-files-loaded", performAnalyzeAll, clearCache);
          self.showFilesDialog = false;
        })
      }
    },
    setSibsInCohortModel: function() {
      let self = this;
      let affectedSibList = [];
      if (self.affectedSibs && self.affectedSibs.length > 0) {
        affectedSibList = self.possibleSibs.filter(function(possibleSib) {
          return self.affectedSibs && self.affectedSibs.indexOf(possibleSib.sample) >= 0;
        })
      }
      let unaffectedSibList = [];
      if (self.unaffectedSibs && self.unaffectedSibs.length > 0) {
        unaffectedSibList = self.possibleSibs.filter(function(possibleSib) {
          return self.unaffectedSibs.indexOf(possibleSib.sample) >= 0;
        })
      }
      return self.cohortModel.promiseSetSibs(affectedSibList, unaffectedSibList)
    },
    onClose:  function() {
      let self = this;
      self.$emit("on-close");
      self.showFilesDialog = false;
    },
    onModeChanged: function() {
      if (this.mode == 'trio' && this.cohortModel.getCanonicalModels().length < 3 ) {
        this.$emit('isTrio', true);
        this.promiseInitMotherFather();
      } else if (this.mode == 'single' && this.cohortModel.getCanonicalModels().length > 1) {
        this.$emit('isTrio', false)
        this.removeMotherFather();
      }

      this.validate();
    },
    onLoadDemoData: function() {
      let self = this;
      this.$emit('isDemo', true);
      self.isDemo = true;

      self.buildName = self.cohortModel.genomeBuildHelper.getCurrentBuildName();

      if (self.mode == 'single') {
        self.mode = 'trio';
      }

      let p;
      if (self.cohortModel.getCanonicalModels().length < 3 ) {
        p = self.promiseInitMotherFather();
      } else {
        p = Promise.resolve();
      }
      p.then(function() {
        self.cohortModel.demoModelInfos[self.demoAction].forEach(function(modelInfo) {
          let rel = modelInfo.relationship;
          self.modelInfoMap[rel] = modelInfo;
        })
        self.cohortModel.getCanonicalModels().forEach(function(model) {
           self.promiseSetModel(model)
           .then(function() {

            })
           .catch(function(error) {
              self.$emit("on-files-load-error", error)
            })
        })
      })
      .catch(function(error) {
        self.$emit("on-files-load-error", error)
      })
    },
    onSampleError: function(error) {
      this.$emit('on-files-load-error', error)
    },
    promiseSetModel: function(model) {
      let self = this;
      return new Promise(function(resolve, reject) {
        var theModel = model;
        var theModelInfo = self.modelInfoMap[theModel.relationship];
        theModelInfo.model = theModel;
        theModel.promiseLoadVcfUrl(theModelInfo.vcf, null)
        .then( function(sampleNames) {
          self.validate();
          theModelInfo.samples = sampleNames;
          if (theModel.relationship == 'proband') {
            self.possibleSibs = sampleNames.map(function(sampleName) {
              return {sample: sampleName, sex: null}
            });
          }
          self.$refs.sampleDataRef.forEach(function(ref) {
            if (ref.modelInfo.relationship == theModel.relationship) {
              theModel.sampleName = theModelInfo.sample;
              ref.updateSamples(sampleNames, theModelInfo.sample);
              theModel.name = theModel.sampleName;
              self.validate();
            }
          })
          theModel.promiseLoadBamUrl(theModelInfo.bam, null)
          .then(function() {
            self.validate();
            resolve();
          })
          .catch(function(error) {
            reject(error);
          })
        })
        .catch(function(error) {
          reject(error)
        })
      })
    },
    validate: function() {
      this.isValidFileCombo = false;
      this.isValid = false;
      this.cohortModel.isLoaded = false;
      if (this.mode == 'single') {
        if (this.modelInfoMap.proband && this.modelInfoMap.proband.model.isReadyToLoad()) {
          this.isValid = true;
          this.cohortModel.isLoaded = true;
        }
      } else {
        if (this.modelInfoMap.proband && this.modelInfoMap.proband.model && this.modelInfoMap.proband.model.isReadyToLoad()
            && this.modelInfoMap.mother && this.modelInfoMap.mother.model && this.modelInfoMap.mother.model.isReadyToLoad()
            && this.modelInfoMap.father && this.modelInfoMap.father.model && this.modelInfoMap.father.model.isReadyToLoad()) {

            this.isValid = true;
            this.cohortModel.isLoaded = true;            
        
        }
      }
      if (this.cohortModel.isValidAlignmentsOnly()) {
        this.isValidFileCombo = true;
      } 

      if (this.isValid && this.cohortModel.isLoaded) {
        this._validateBuild();
      }
    },
/* 
    *  Compare the build specified to the vcf header(s). If the vcf header provides information regarding the build,
    *  (by way of the assembly on the contig header), and it doesn't match the build specified, 
    *  set isValidBuild to false and display a message indicating which samples (if trio) have a discordant build.
    * 
    *  NOTE: This method assumes that validate() has been called and will valid the build if 
    *        isValid is set to true.
    */
    _validateBuild: function() {
      let self = this;
      return new Promise(function(resolve, reject) {
        self.isValidBuild = true;
        self.invalidBuildMessage = '';
        if (self.isValid) {
          self.cohortModel.promiseValidateBuild(self.buildName, self.mode)
          .then(function(data) {
            self.isValidBuild = data.isValidBuild;
            self.invalidBuildMessage = data.message;
            resolve();
          })
          .catch(function(error) {
            reject(error)
          })
        } else {
          resolve();
        }
      })
    },    
    onSamplesAvailable: function(relationship, samples) {
      let self = this;
      if (relationship == 'proband') {
        this.possibleSibs = samples.map(function(sample) {
          return {'sample': sample, 'sex': 'unknown'}
        })
        this.cohortModel.sampleMapSibs.affected.forEach(function(sampleModel) {
          self.possibleSibs.forEach(function(possibleSib) {
            if (possibleSib.sample == sampleModel.sampleName) {
              possibleSib.sex = sampleModel.sex
            }
          })
        })
        this.cohortModel.sampleMapSibs.unaffected.forEach(function(sampleModel) {
          self.possibleSibs.forEach(function(possibleSib) {
            if (possibleSib.sample == sampleModel.sampleName) {
              possibleSib.sex = sampleModel.sex
            }
          })
        })
        if (this.cohortModel.sampleMapSibs.affected && this.cohortModel.sampleMapSibs.affected.length > 0) {
          this.affectedSibs = this.cohortModel.sampleMapSibs.affected.map(function(sampleModel) {
             return sampleModel.sampleName;
          })
        }
        if (this.cohortModel.sampleMapSibs.unaffected && this.cohortModel.sampleMapSibs.unaffected.length > 0) {
          this.unaffectedSibs = this.cohortModel.sampleMapSibs.unaffected.map(function(sampleModel) {
             return sampleModel.sampleName;
          })
        }
      }
      if (samples && samples.length > 0 && this.getModel(relationship)) {
        this.getModel(relationship).samples = samples;
      }
    },
    getModel: function(relationship) {
      var theModel = null;
      if (this.cohortModel) {
        var modelObject = this.cohortModel.sampleMap[relationship];
        if (modelObject) {
          theModel = modelObject.model;
        }
      }
      return theModel;
    },
    init: function() {
      let self = this;
      self.buildName = self.cohortModel.genomeBuildHelper.getCurrentBuildName()
      self.modelInfoMap = {};
      if (self.cohortModel && self.cohortModel.getCanonicalModels().length > 0) {
        self.initModelInfo();
      } else {
        var modelInfo = {};
        modelInfo.relationship = 'proband';
        modelInfo.vcf = null;
        modelInfo.bam = null;
        modelInfo.affectedStatus = 'affected'
        self.cohortModel.promiseAddSample(modelInfo)
        .then(function() {
          self.initModelInfo();
        })
      }
    },
    initModelInfo: function() {
      let self = this;
      self.separateUrlForIndex = false;
      self.cohortModel.getCanonicalModels().forEach(function(model) {
        var modelInfo = self.modelInfoMap[model.relationship];
        if (modelInfo == null) {
          modelInfo = {};
          modelInfo.relationship = model.relationship;
          modelInfo.sex          = model.sex ? model.sex : null;
          modelInfo.vcf          = model.vcf ? model.vcf.getVcfURL() : null;
          modelInfo.tbi          = model.vcf ? model.vcf.getTbiURL() : null;
          modelInfo.bam          = model.bam ? model.bam.bamUri : null;
          modelInfo.bai          = model.bam ? model.bam.baiUri : null;
          modelInfo.sample       = model.getSampleName();
          modelInfo.name         = model.getName();
          modelInfo.samples      = model.sampleNames;
          modelInfo.isAffected   = model.isAffected();
          modelInfo.model        = model;
          if (modelInfo.tbi || modelInfo.bai) {
            self.separateUrlForIndex = true;
          }
          self.$set(self.modelInfoMap, model.relationship, modelInfo);
        }
      })
    },
    promiseInitMotherFather: function() {
      let self = this;

      return new Promise(function(resolve, reject) {
        var modelInfoMother = {};
        modelInfoMother.relationship = 'mother';
        modelInfoMother.sex = "female";
        modelInfoMother.vcf = null;
        modelInfoMother.bam = null;
        modelInfoMother.affectedStatus = 'unaffected'
        self.cohortModel.promiseAddSample(modelInfoMother)
        .then(function() {
          var modelInfoFather = {};
          modelInfoFather.relationship = 'father';
          modelInfoFather.sex = "male"
          modelInfoFather.vcf = null;
          modelInfoFather.bam = null;
          modelInfoFather.affectedStatus = 'unaffected'

            self.cohortModel.promiseAddSample(modelInfoFather)
            .then(function() {

              self.initModelInfo();
              resolve();

            })
            .catch(function(error) {
              reject(error);
            })
        })
        .catch(function(error) {
          reject(error);
        })

      })
    },
    removeMotherFather: function() {
      let self = this;
      delete self.modelInfoMap.mother;
      delete self.modelInfoMap.father;
      self.cohortModel.removeSample("mother");
      self.cohortModel.removeSample("father");
    }
  },
  computed: {
    buildList: function() {
      if (this.speciesName && this.cohortModel.genomeBuildHelper) {
        return this.cohortModel.genomeBuildHelper.speciesToBuilds[this.speciesName].map(function(gb) {
          return gb.name;
        })
      } else {
        return [];
      }
    },
    clazzAttention: function() {
      if (!this.buildName) {
        return 'attention';
      }
      else {
        return '';
      }
    }
  },
  created: function() {
    let self = this;

  },
  mounted: function() {
    if (this.cohortModel) {
      this.speciesName =  this.cohortModel.genomeBuildHelper.getCurrentSpeciesName();
      this.speciesList =  this.cohortModel.genomeBuildHelper.speciesList.map(function(sp) {
        return sp.name;
      }).filter(function(name) {
        return name == 'Human';
      });
    }



  }
}
</script>
