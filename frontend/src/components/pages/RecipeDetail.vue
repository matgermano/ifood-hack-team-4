<template>
  <Dialog
    ref="dialog"
    :dialog="dialog"
    @close="closeDialog"
    @update:dialog="dialog = $event"
  >
    <template v-slot:content>
      <div v-if="step === 1">
        <div v-if="!isMobile" class="stepper">
          <Stepper :step="step" @update:step="step = $event"/>
        </div>
        <v-container :class="isMobile ? 'container-mobile' : 'container'">
          <div :class="isMobile ? 'checkout-price mobile' : 'checkout-price'">
            <span class="checkout">Checkout</span>
            <span>Pedido: R${{total}}</span>
          </div>
          <div :class="isMobile ? 'container-details mobile' : 'container-details'">
            <div :class="isMobile ? 'recipe-detail mobile' : 'recipe-detail'">
              <span>{{recipe.title}}</span>
              <div class="user-time">
                <div>
                  <v-icon>mdi-account</v-icon>
                  <v-icon>mdi-account</v-icon>
                </div>
                <span>30 min</span>
              </div>
              <v-img class="recipe-img" :src="recipe.image"></v-img>
            </div>
            <div :class="!isMobile ? 'ingredients': 'ingredients-mobile'">
              <span>{{recipe.description}}</span>
              <p class="ingredients-title">Confira os ingredientes: </p>
              <div :class="isMobile ? 'ingredients-check-mobile' : 'ingredients-check'" v-for="ingredient in ingredients">
                <v-checkbox
                  v-model="ingredient.checked"
                  class="checkbox"
                  :label="ingredient.title"
                />
                <span>R$ {{ingredient.price}}</span>
              </div>
            </div>
          </div>
          <span class="choose-market">Escolha seu mercado:</span>
          <v-autocomplete
            v-model="market"
            :items="markets"
            filled
            chips
            item-text="name"
            item-value="name"
          >
            <template v-slot:selection="data">
              <v-chip
                v-bind="data.attrs"
                :input-value="data.selected"
                close
                text-color="white"
                @click="data.select"
                @click:close="remove(data.item)"
              >
                <v-avatar left>
                  <v-img :src="data.item.logo"></v-img>
                </v-avatar>
                {{ data.item.name }}
              </v-chip>
            </template>
            <template v-slot:item="data">
                <v-list-item-avatar>
                  <img :src="data.item.logo">
                </v-list-item-avatar>
                <v-list-item-content>
                  <v-list-item-title v-html="data.item.name"></v-list-item-title>
                </v-list-item-content>
            </template>
          </v-autocomplete>
        </v-container>
      </div>
      <div v-else class="complete">
        <v-icon v-if="showFirstIcon" class="complete-icon">mdi-silverware-fork-knife </v-icon>
        <v-icon v-else class="complete-icon">mdi-check-circle-outline </v-icon>
        <span class="complete-main-message">Quase lá, seu pedido está a caminho</span>
        <span class="complete-download-message">Veja como preparar esta receita</span>
        <v-btn
          color="#EA1D2C"
          dark
          class="text-capitalize"
          @click="conclude"
        >
          Baixar Receita
        </v-btn>
      </div>
    </template>
    <template v-if="step === 1" v-slot:footer>
      <span class="mr-6">
        Serve: {{count}}
        <v-icon @click="decrease" color="black" class="ml-2 plus-icon">mdi-minus-circle</v-icon>
        <v-icon @click="count++" color="black" class="ml-2 plus-icon">mdi-plus-circle</v-icon>
      </span>
      <v-btn
        color="#EA1D2C"
        dark
        class="text-capitalize mr-10"
        @click="step = '2'"
      >
        Fechar Pedido
      </v-btn>
    </template>
  </Dialog>
</template>

<script>
  import Dialog from '@/components/layout/Dialog.vue'
  import Stepper from '@/components/layout/Stepper.vue'

  export default {
    name: 'RecipeDetail',

    components: {
      Dialog,
      Stepper
    },

    props: {
      dialog: {
        required: true,
        default: false
      },

      recipe: {
        required: true
      }
    },

    watch: {
      step(value) {
        if (value === "2") {
          this.showFirstIcon = true

          setTimeout(() => {
            this.showFirstIcon = false
          }, 2000)
        }
      },

      dialog(value) {
        if (value) {
          this.ingredients = this.recipe.ingredients
          this.setIngredientsAsChecked()

          let markets = []
          this.ingredients.forEach(ingredient => {
            console.log(ingredient.markets)
            markets = markets.concat(ingredient.markets)
          })

          this.markets = markets
        }
      }
    },

    data: () => ({
      step: 1,
      ingredients: [
        {title: '', price: 0, checked: true},
      ],
      markets: [
        {title: '', image: ""}
      ],
      check: true,
      market: {},
      count: 1,
      showFirstIcon: false
    }),

    computed: {
      internalDialog: {
        get: function() {
          return this.dialog
        },
        set: function(value) {
          this.$emit('close', value)
        }
      },

      total() {
        const ingredients = this.ingredients.filter(ingredient => ingredient.checked)

        if (ingredients.length) {
          return (ingredients.reduce((a, b) => ({price: a.price + b.price})).price * this.count).toFixed(2)
        }

        return 0
      },

      isMobile() {
        return this.$vuetify.breakpoint.xs || this.$vuetify.breakpoint.sm
      }
    },

    methods: {
      remove () {
        this.market = null
      },

      conclude() {
        this.download()
        this.closeDialog()
      },

      closeDialog() {
        this.internalDialog = false
        this.count = 1
        this.market = null
        this.setIngredientsAsChecked()

        setTimeout(() => {
          this.step = 1
        }, 400)
      },

      setIngredientsAsChecked() {
        this.ingredients.forEach(ingredient => ingredient.checked = true)
      },

      download() {
        var element = document.createElement('a')
        element.setAttribute('href', 'data:text/plain;charset=utf-8,')
        element.setAttribute('download', 'receita.pdf');

        element.style.display = 'none'
        document.body.appendChild(element)

        element.click()

        document.body.removeChild(element)
      },

      decrease() {
        if (this.count > 1) {
          this.count--
        }
      }
    }
  }
</script>

<style lang="sass" scoped>
  .stepper
    display: flex
    justify-content: center

  .container
    padding:0px 50px

  .container-mobile
    padding: unset

  .container-details
    display: flex
    margin-bottom: 20px

  .checkout-price
    display: flex
    justify-content: space-between
    align-items: center
    font-size: 18px
    margin-bottom: 10px

  .checkout
    font-size: 23px

  .recipe-detail
    width: 40%
    display: flex
    flex-direction: column

  .recipe-img
    width: 245px
    height: 305px
    object-fit: cover
    align-self: center
    border-radius: 18px

  .user-time
    display: flex
    justify-content: space-between

  .ingredients
    margin-top: 40px
    margin-left: 40px

  .ingredients-mobile
    margin-top: 40px

  .ingredients-title
    font-size: 18px
    margin-top: 20px

  ::v-deep .v-messages
    display: none !important

  .checkbox
    margin-top: 0px

  .choose-market
    font-size: 18px

  .plus-icon
    font-size: 20px
    cursor: pointer

  .complete
    flex-direction: column
    height: 450px
    display: flex
    justify-content: center
    overflow: hidden
    align-items: center

  .complete-icon
    font-size: 100px

  .complete-main-message
    font-size: 30px
    margin-top: 50px

  .complete-download-message
    font-size: 22px
    margin: 20px

  .ingredients-check
    width: 80%
    display: flex
    align-items: center
    justify-content: space-between

  .ingredients-check-mobile
    display: flex
    align-items: center
    justify-content: space-between


  .mobile
    flex-direction: column
    width: 100%
    padding:0px
</style>
