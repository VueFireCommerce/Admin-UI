<template>
  <div>
    <v-card color="grey lighten-3" class="mb-12">
      <v-container>
        <v-form class="ma-4" ref="stage">
          <v-row>
            <v-col cols="12" md="6">
              <v-text-field
                v-model="currentProductDetail.name"
                label="Product Name"
                :rules="nameRules"
                required
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="6">
              <v-select
                v-model="currentProductDetail.type"
                :items="categories"
                :rules="nameRules"
                label="Select a category"
              ></v-select>
            </v-col>
          </v-row>
        </v-form>
      </v-container>
    </v-card>
    <v-btn color="primary" :loading="loading" @click="submit">Continue</v-btn>
    <v-btn text @click="$emit('cancel')">Cancel</v-btn>
  </div>
</template>

<script lang="ts">
import vue from "vue";
import { Component, Prop } from "vue-property-decorator";
import { IProduct, ProductType } from "../../../../Common/IProducts";
import * as firebase from "firebase/app";
import "firebase/firestore";

@Component
export default class Stage1 extends vue {
  loading = false;
  final: IProduct = {} as IProduct;

  categories = ["tea"];

  nameRules: Function[] = [(v: string) => !!v || "This is required"];

  get currentProductDetail() {
    return this.$store.getters.getCurrentProduct as IProduct;
  }

  $refs!: {
    stage: HTMLFormElement;
  };

  validateStage(): boolean {
    return this.$refs.stage.validate();
  }
  submit() {
    if (this.validateStage()) {
      const that = this;

      this.final.name = this.currentProductDetail.name;
      this.final.type = this.currentProductDetail.type as ProductType;

      this.loading = true;
      const db = firebase.firestore();

      if (this.isExistingProduct == false) {
        db.collection("Product")
          .add(this.final)
          .then(docRef => {
            that.$store.commit("setDocRef", docRef.id);
            that.loading = false;
            that.$emit("submit");
          })
          .catch(err => {
            console.warn(err);
            that.loading = false;
          });
      } else {
        db.collection("Product")
          .doc(this.currentProduct)
          .update(this.final)
          .then(() => {
            that.loading = false;
            that.$emit("submit");
          })
          .catch(err => {
            console.warn(err);
            that.loading = false;
          });
      }
    }
  }
  get currentProduct(): string {
    return this.$store.getters.getDocRef;
  }
  get isExistingProduct(): boolean {
    return !!this.currentProduct;
  }
}
</script>
