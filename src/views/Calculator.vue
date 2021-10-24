<template>
  <v-container>
    <v-row justify="center" class="my-auto">
      <v-col cols="12" md="6">
        <v-toolbar color="blue" dark flat>
          <v-icon>mdi-calculator</v-icon>
          <v-card-title> Calculator </v-card-title>
        </v-toolbar>
        <v-card max-height="520" style="overflow: auto">
          <v-container>
            <v-form v-model="isCalcFormValid" ref="calculatorForm" class="my-4">
              <v-select
                v-model="vatPercentage"
                :items="vats"
                item-value="value"
                dense
                outlined
                clearable
                label="VAT rate"
                :rules="inputsValidationRules.vatPercentage"
              ></v-select>

              <v-text-field
                v-model="goodTitle"
                outlined
                dense
                clearable
                label="Title"
                :rules="inputsValidationRules.goodTitle"
              >
              </v-text-field>

              <v-text-field
                v-model.number="goodPrice"
                outlined
                dense
                clearable
                label="Price"
                :rules="inputsValidationRules.goodPrice"
              >
              </v-text-field>
              <v-radio-group
                v-model="vatOperator"
                row
                :rules="inputsValidationRules.vatOperator"
              >
                <v-radio label="Add VAT" value="add"></v-radio>
                <v-radio label="Remove VAT" value="remove"></v-radio>
              </v-radio-group>
              <v-row v-if="isCalcFormValid" class="mb-4">
                <v-divider></v-divider>
                <v-col cols="6" lg="3">
                  <v-card height="100%">
                    <v-card-text class="font-weight-bold">
                      Original cost
                    </v-card-text>
                    <v-card-text>{{ goodPrice }} zł</v-card-text>
                  </v-card>
                </v-col>
                <v-col cols="6" lg="3">
                  <v-card height="100%">
                    <v-card-text class="font-weight-bold"> VAT % </v-card-text>
                    <v-card-text> {{ vatPercentage }}%</v-card-text>
                  </v-card>
                </v-col>
                <v-col cols="6" lg="3">
                  <v-card height="100%">
                    <v-card-text class="font-weight-bold">
                      VAT Price
                    </v-card-text>
                    <v-card-text>
                      {{ `${vatOperator === "add" ? "+" : "-"}` }}
                      {{ vatPrice }} zł</v-card-text
                    >
                  </v-card>
                </v-col>
                <v-col cols="6" lg="3">
                  <v-card height="100%">
                    <v-card-text class="font-weight-bold">
                      Net Price
                    </v-card-text>
                    <v-card-text>{{ netPrice }} zł</v-card-text>
                  </v-card>
                </v-col>
              </v-row>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn
                  @click="addToGoodsList"
                  color="success"
                  :disabled="!isCalcFormValid"
                >
                  <span class="white--text"> Add to my list</span>
                </v-btn>
                <v-btn @click="resetForm" color="error" dark>Сlear</v-btn>
              </v-card-actions>
            </v-form>
          </v-container>
        </v-card>
      </v-col>
      <v-col cols="12" md="6">
        <v-toolbar color="blue" dark flat>
          <v-icon>mdi-cart</v-icon>
          <v-card-title> My purchases </v-card-title>
        </v-toolbar>
        <v-card height="520" class="d-flex flex-column" style="overflow: auto">
          <v-card-text v-if="!goods.length" fluid>
            You have no goods in your list yet
          </v-card-text>
          <v-card-text v-else>
            <v-list v-for="(good, index) in goods" :key="index">
              <v-list-item>
                <v-list-item-avatar>
                  <v-icon color="success"> mdi-check </v-icon>
                </v-list-item-avatar>

                <v-list-item-content>
                  <v-list-item-title
                    v-text="good.title || 'no name'"
                  ></v-list-item-title>
                </v-list-item-content>

                <v-list-item-action>
                  <v-row>
                    <v-menu
                      v-model="good.goodDetailsInfoPopUp"
                      :close-on-content-click="false"
                      :nudge-width="200"
                      offset-x
                    >
                      <template v-slot:activator="{ on }">
                        <v-btn icon v-on="on">
                          <v-icon color="grey lighten-1">
                            mdi-information
                          </v-icon>
                        </v-btn>
                      </template>
                      <v-card class="pa-4">
                        <div>
                          <span class="mr-2 font-weight-bold">
                            Original price:
                          </span>
                          <span>{{ good.goodPrice }} zł</span>
                        </div>
                        <div>
                          <span class="mr-2 font-weight-bold">VAT %: </span>
                          <span>{{ good.vatPercentage }}%</span>
                        </div>
                        <div>
                          <span class="mr-2 font-weight-bold">VAT Price: </span>
                          {{ `${good.vatOperator === "add" ? "+" : "-"}` }}
                          <span> {{ good.vatPrice }} zł</span>
                        </div>
                        <div>
                          <span class="mr-2 font-weight-bold">
                            Net Price:
                          </span>
                          <span>{{ good.netPrice }} zł</span>
                        </div>
                      </v-card>
                    </v-menu>
                    <v-btn icon @click="deleteFromGoodsList(index)">
                      <v-icon color="grey lighten-1"> mdi-delete </v-icon>
                    </v-btn>
                  </v-row>
                </v-list-item-action>
              </v-list-item>
              <v-divider></v-divider>
            </v-list>
          </v-card-text>
          <v-spacer></v-spacer>
          <div v-if="goods.length">
            <v-divider></v-divider>
            <v-card-text>
              <span class="font-weight-bold">Total gross: </span>
              <span>{{ totalGross }} zł</span>
            </v-card-text>
            <v-card-text class="pt-0">
              <span class="font-weight-bold">Total net: </span>
              <span>{{ totalNet }} zł</span>
            </v-card-text>
          </div>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "Calculator",

  data() {
    return {
      isCalcFormValid: false,
      vats: [
        { text: "23% (standart)", value: 23 },
        { text: "8% (reduced)", value: 8 },
        { text: "5% (reduced)", value: 5 },
      ],
      vatPercentage: null,
      vatOperator: null,
      goodTitle: "",
      goodPrice: null,
      goods: [],
      inputsValidationRules: {
        vatOperator: [(v) => !!v || "Choose option"],
        vatPercentage: [(v) => !!v || "Choose VAT percentage"],
        goodTitle: [(v) => !!v || "Enter title"],
        goodPrice: [
          (v) => !!v || "Enter a price",
          (v) => typeof v === "number" || "Price must be a number",
        ],
      },
    };
  },

  computed: {
    vatPrice() {
      return Number(((this.goodPrice / 100) * this.vatPercentage).toFixed(2));
    },

    netPrice() {
      switch (this.vatOperator) {
        case "add":
          return Number((this.goodPrice + this.vatPrice).toFixed(2));

        case "remove":
          return Number((this.goodPrice - this.vatPrice).toFixed(2));
      }
    },

    computedGoodInfo() {
      return {
        title: this.goodTitle,
        goodPrice: this.goodPrice,
        vatPercentage: this.vatPercentage,
        vatOperator: this.vatOperator,
        vatPrice: this.vatPrice,
        netPrice: this.netPrice,
        goodDetailsInfoPopUp: false,
      };
    },

    totalGross() {
      let result = 0;
      this.goods.forEach(({ goodPrice }) => (result += goodPrice));
      return result;
    },

    totalNet() {
      let result = 0;
      this.goods.forEach(({ netPrice }) => (result += netPrice));
      return result;
    },
  },

  methods: {
    addToGoodsList() {
      this.goods.push(this.computedGoodInfo);
      this.resetForm();
    },

    deleteFromGoodsList(index) {
      this.goods = [...this.goods].filter((_, i) => i !== index);
    },

    resetForm() {
      this.$refs.calculatorForm.reset();
    },
  },
};
</script>
