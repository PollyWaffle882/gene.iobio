/*
 * RankedVariantsMenu
 *
 */
<style lang="sass">

@import ../../../assets/sass/variables


#ranked-variants-menu-close-button
  right: 1px
  top: 1px
  position: absolute
  margin: 0px
  padding: 0px
  min-width: 20px


  .btn__content, .v-btn__content
    padding: 4px

#ranked-variants-body
  display: flex
  flex-direction: column


  #ranked-variants-matrix
    max-width: 370px
    overflow-x:  auto
    box-shadow: none !important
    -webkit-box-shadow: none !important

    #feature-matrix-note


  #ranked-variant-info
    margin-left: 10px
    width: 400px

    margin-bottom: 10px

    #ranked-variant-header
      margin-top: 0px
      margin-bottom: 5px

    .row
      padding-left: 20px !important
      padding-right: 20px !important
      font-size:  12px !important
      padding-bottom: 0px !important

      .clinvar-submission
        app-icon
          display: none !important
        span
          padding-left: 0px !important

        .tooltip-clinvar-link
          color:  $text-color !important
          cursor: normal !important


  .matrix-note-text
    margin-top: -7px
    color: $text-color
    font-style: italic

  #feature-matrix-note
    padding-left: 50px
    font-size: 12px
    margin-top: -15px

#ranked-variants-menu
  vertical-align: bottom
  padding-bottom: 4px
  padding-top: 2px


  #ranked-variants-menu-button
    margin: 0px 0px 5px 0px
    font-size: 13px
    height: 26px
    margin-bottom: 0px
    margin-top: 0px
    max-width: 50px
    min-width: 50px


    .btn__content, .v-btn__content
      padding-left: 5px
      padding-right: 5px
      font-weight: 500
      color: $link-color


      i.material-icons
        color: $link-color
        font-size: 20px
        padding-right: 3px

</style>




<template>

    <v-menu id="ranked-variants-menu"
      offset-y
      :close-on-content-click="false"
      :nudge-height="500"
      top
      v-model="showRankedVariantsMenu">

        <v-btn id="ranked-variants-menu-button" flat
         slot="activator">
          <v-icon >menu</v-icon>
          List
        </v-btn>

        <div style="width:420px">
          <v-btn id="ranked-variants-menu-close-button" @click="showRankedVariantsMenu = false" flat>
             <v-icon>close</v-icon>
          </v-btn>


          <div id="ranked-variants-body">



            <feature-matrix-card
              id="ranked-variants-matrix"
              ref="featureMatrixCardRef"
              :isEduMode="isEduMode"
              :isBasicMode="isBasicMode"
              :isSimpleMode="isSimpleMode"
              :featureMatrixModel="featureMatrixModel"
              :selectedGene="selectedGene"
              :selectedTranscript="selectedTranscript"
              :selectedVariant="selectedVariant"
              relationship="PROBAND"
              :width="300"
              @cohort-variant-click="onRankedVariantClick"
              @cohort-variant-hover="onRankedVariantHover"
              @cohort-variant-hover-end="onRankedVariantHoverEnd"
              @variant-rank-change="featureMatrixModel.promiseRankVariants(cohortModel.getModel('proband').loadedVariants);"
              >
             </feature-matrix-card>

            <div id="ranked-variant-info" v-if="currentVariant && currentVariantInfo">
              <div id="ranked-variant-header" >
                <span class="pl-1">{{ currentVariant.type ? currentVariant.type.toUpperCase() : "" }}</span>
                <span class="pl-1">{{ currentVariantInfo.coord }}</span>
                <span class="pl-1 refalt">{{ refAlt  }}</span>
              </div>

              <v-flex v-if="!isBasicMode && currentVariant.inheritance != '' && currentVariant.inheritance.indexOf('n/a') == -1 ">
                <v-layout row class="">
                   <v-flex xs4 class="field-label">Inheritance</v-flex>
                   <v-flex id="inheritance" xs8 class="field-value">
                     <app-icon :icon="currentVariant.inheritance" height="16" width="16">
                     </app-icon>
                     {{ currentVariant.inheritance == 'denovo' ? 'de novo' : currentVariant.inheritance }}
                   </v-flex>
                </v-layout>
              </v-flex>
              <v-flex>
                <v-layout row>
                   <v-flex xs4  class="field-label">Impact</v-flex>
                   <v-flex xs8  class="field-value" v-html="impactAndConsequence"></v-flex>
                </v-layout>
              </v-flex>

              <v-flex >
                <v-layout row  v-if="currentVariantInfo.clinvarLinks.length > 0">
                   <v-flex xs4 class="field-label">Clinvar</v-flex>
                   <v-flex xs8   class="field-value">
                    <span class="clinvar-submission" v-for="clinvarLink in currentVariantInfo.clinvarLinks"
                     :key="clinvarLink.key">
                       <app-icon width="14" height="14" icon="clinvar" :significance="clinvarLink.significance">
                       </app-icon>

                       <span style="padding-left: 5px" v-html="clinvarLink.link">
                       </span>

                    </span>
                   </v-flex>
                </v-layout>


              </v-flex>

              <v-flex  v-if="currentVariantInfo.revel != '' && currentVariantInfo.revel != null && !isBasicMode" >
                <v-layout row class="">
                   <v-flex xs4 class="field-label revel">REVEL</v-flex>
                   <v-flex xs8>
                      {{ currentVariantInfo.revel }}
                   </v-flex>
                </v-layout>
              </v-flex>

              <v-flex>
                <v-layout  row>
                   <v-flex xs4 class="field-label">gnomAD</v-flex>
                   <v-flex xs8 class="field-value" v-html="afGnomAD"></v-flex>
                </v-layout>
            </v-flex>

            </div>
          </div>
        </div>

    </v-menu>


  </v-card>



</template>

<script>


import FeatureMatrixCard    from "../viz/FeatureMatrixCard.vue"
import AppIcon              from "../partials/AppIcon.vue"


export default {
  name: 'ranked-variants-menu',
  components: {
    FeatureMatrixCard,
    AppIcon
  },
  props: {
    isEduMode: null,
    isBasicMode: null,
    isSimpleMode: null,
    variantTooltip: null,
    selectedGene: {},
    selectedTranscript: {},
    selectedVariant: null,
    info: null,
    featureMatrixModel: null

  },




  data() {
    let self = this;
    return {
      showRankedVariantsMenu: false,
      currentVariant: null,
      currentVariantInfo: null

    }
  },


  methods: {

    setCurrentVariant(variant) {
      this.currentVariant = variant;
      if (this.currentVariant) {
        this.currentVariantInfo =  this.globalApp.utility.formatDisplay(this.currentVariant, this.featureMatrixModel.cohort.translator, this.isEduMode)
      } else {
        this.currentVariantInfo = null;
      }
    },

    onRankedVariantClick: function(variant, sourceComponent, sourceRelationship) {
      this.setCurrentVariant(variant);
      this.$emit("ranked-variant-click", variant, sourceComponent, sourceRelationship);
      this.showRankedVariantsMenu = false;
    },
    onRankedVariantHover: function(variant, sourceComponent) {
      this.setCurrentVariant(variant);
      this.$emit("ranked-variant-hover", variant, sourceComponent);
    },
    onRankedVariantHoverEnd: function(sourceVariantCard) {
      //this.setCurrentVariant(null);
      this.$emit("ranked-variant-hover-end", sourceVariantCard);
    },


  },


  filters: {
  },

  computed: {

    refAlt: function() {
      let self = this;
      var refAlt = "";
      if (self.selectedGene && self.selectedGene.strand && self.currentVariant) {
        if (self.isEduMode) {
          if (self.selectedGene.strand == "-") {
            refAlt = self.globalApp.utility.switchGenotype(self.currentVariant.eduGenotype)
          } else {
            refAlt = self.currentVariant.eduGenotype;
          }
        } else {
          refAlt =   self.currentVariantInfo.refalt;
        }
      }
      return refAlt;
    },
    afGnomAD: function(af) {
      if (this.currentVariant.gnomad.AF == null) {
        return "<span>"
        + "0%"
        + "</span>";
      } else {
        return "<span>"
        + this.globalApp.utility.percentage(this.currentVariant.gnomad.AF)
        + "</span>";
      }

    },
    impactAndConsequence: function() {
      return "<span class=>"
       + this.currentVariantInfo.vepImpact
       + " - "
       +  this.currentVariantInfo.vepConsequence
       + "</span>"
    },
  },

  watch: {
    showRankedVariantsMenu: function() {
      if (this.showRankedVariantsMenu) {
        this.setCurrentVariant(this.selectedVariant);
        if (this.currentVariant && this.$refs.featureMatrixCardRef) {
          this.$refs.featureMatrixCardRef.selectVariant(this.currentVariant);
        }
      }
    }

  },




  mounted: function() {

  },

  created: function() {
  }



}
</script>
