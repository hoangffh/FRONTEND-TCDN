<template>
  <div
    ref="base_cbb"
    :class="['base-cbb-tb', { disable: isDisable }]"
    :style="{ width: width, minHeight: minHeight }"
  >
    <div class="input__wrap" ref="input">
      <input
        :style="[
          { height: minHeight },
          notAllowCreate ? { paddingRight: '32px' } : {}
        ]"
        :class="[ isErr ?'m-input-error' : '']"
        :title="textSearch"
        ref="input_search"
        class="cbb__input"
        :disabled="isDisable ? true : false"
        @keydown="handleKeyDownInput"
        v-model="textSearch"
        @input="handleOnInput"
        :tabindex="tabindex"
      />
      <div class="cbb__icon__add-wrap" @click="handleClikAdd" v-show="!notAllowCreate">
        <div class="cbb__icon__add"></div>
      </div>
      <div class="cbb__icon-wrap" @click="handleClickDisplayIcon">
        <div class="cbb__icon-hide" v-show="!isShowList"></div>
        <div class="cbb__icon-show" v-show="isShowList"></div>
      </div>
    </div>
  </div>
  <!-- list option of cbb -->
  <div
    class="cbb__tb__list"
    ref="cbb"
    v-show="isShowList"
    :style="{
      top: `${dropDownY}px`,
      left: `${dropDownX}px`,
      minWidth: `${inputWidth}px`,
      top: `${dropDownYValue}px`,
      left: `${dropDownXValue}px`,
    }"
    @scroll="handleScrollListOptions"
  >
    <table :style="{ width: '100%' }">
      <thead class="cbb__list__item-header" ref="header">
        <tr>
          <th
            class="cbb__item__col"
            v-for="(item, index) in infor"
            :key="index"
            :style="{ width: item.width }"
          >
            <div :style="{ width: item.width }">{{ item.display }}</div>
          </th>
        </tr>
      </thead>
      <tbody ref="listCbb">
        <tr
          v-for="(dataItem, dataIndex) in dataChanged"
          :key="dataIndex"
          @click="handleClickOption(dataItem)"
          :class="[
            dataItem[fieldValue] == modelValue ? 'active' : '',
            `cbbItem-${dataIndex}`,
          ]"
        >
          <td
            class="cbb__item__col"
            v-for="(item, index) in infor"
            :key="index"
          >
            <div :style="{ width: item.width }" :title="dataItem[item.name]">
              {{ dataItem[item.name] }}
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
// import { mapMutations } from "vuex";
export default {
  name: "base-combobox-table",
  props: [
    "width",
    "minHeight",
    "data",
    "infor",
    "fieldValue",
    "fieldDisplay",
    "tabindex",
    "modelValue",
    "notAllowCreate",
    "isDisable",
    "focus",
    "size",
    "dropDownXValue",
    "dropDownYValue",
    "isErr",
    "field",
  ],

  data() {
    return {
      isShowList: false,
      textSearch: "",
      dropDownX: null,
      dropDownY: null,
      inputWidth: 0,
      dataChanged: [],
    };
  },
  emits: ["handleScroll", "update:modelValue", "handleTab"],

  computed: {},
  watch: {
    isErr(value) {
      console.log(value)
    },
    isShowList(value) {
      if (value) {
        document.addEventListener("click", this.focusOutSide);
      } else {
        document.removeEventListener("click", this.focusOutSide);
      }
    },
    //set data s??? d??ng ????? hi???n th???
    data(value) {
      this.dataChanged = [...value];
    },
    //khi modelValue thay ?????i => bind text v??o input
    modelValue(value) {
      if (!value) {
        this.textSearch = "";
      } else {
        const fieldSl = this.data.find((itemItem) => {
          return itemItem[this.fieldValue] == value;
        });
        if (fieldSl) {
          this.textSearch = fieldSl[this.fieldDisplay];
        } else {
          this.textSearch = "";
        }
      }
    },
    //focus v??o input n???u prop focus = true
    focus(value) {
      if (value) {
        this.$nextTick(() => this.$refs.input_search.focus());
      }
    },
  },

  mounted() {
    if (this.modelValue) {
      //set text v??o input khi modelValue thay ?????i
      const res = this.data.find((item) => {
        return item[this.fieldValue] == this.modelValue;
      });
      if (res) {
        this.textSearch = res[this.fieldDisplay];
      }
    }
  },

  methods: {
    handleClikAdd()
    {
      this.$emit("handleClikAdd")
    },
    // ...mapMutations([]),
    //emit h??m scroll khi scroll ?????n cu???i
    handleScrollListOptions(e) {
      console.log(this.size);
      if (e.target.scrollHeight - e.target.scrollTop <= 200) {
        this.$emit("handleScroll", Number(this.size) + 100);
      }
    },
    //Khi click v??o icon show
    handleClickDisplayIcon() {
      //n???u ??ang kh??ng ??? ch??? ????? xem
      if (!this.isDisable) {
        //Set v??? tr?? c???a list
        const coordinates = this.$refs.input_search.getBoundingClientRect();
        const inputHeight = this.$refs.input_search.offsetHeight;
        this.inputWidth = this.$refs.input_search.offsetWidth;
        this.dropDownX = coordinates.x;
        if (coordinates.y > 400) {
          this.dropDownY = coordinates.y - 200;
        } else {
          this.dropDownY = coordinates.y + inputHeight;
        }
        //l???y data fake
        if (!this.isShowList) {
          this.dataChanged = this.data;
        }
        this.isShowList = !this.isShowList;
      }
    },
    //h??m x??? l?? click outside
    focusOutSide(e) {
      if (
        !e.target.classList.contains("cbb__icon-wrap") &&
        !e.target.classList.contains("cbb__icon-hide") &&
        !e.target.classList.contains("cbb__icon-show")
      ) {
        this.isShowList = false;
      }
    },
    //emit modelValue khi click item
    handleClickOption(item) {
      const fieldSl = this.infor.find((iteminfor) => {
        return iteminfor.isInputDisplay;
      })?.name;
      this.textSearch = item[fieldSl];

      this.$emit("update:modelValue", item[this.fieldValue]);
      this.$emit('onChangeValue', this.textSearch, this.field)
      this.isShowList = false;
    },
    //X??? l?? khi value input thay ?????i
    handleOnInput() {
      this.$emit('onChangeValue', this.textSearch, this.field)
      this.isShowList = true;
      if (!this.textSearch) {
        //N???u value input == null => emit l???i modelValue null v?? reset data fake
        this.$emit("update:modelValue", null);
        
        this.dataChanged = this.data;
      } else {
        //else => show item theo value input
        this.dataChanged = this.data.filter((item) => {
          const isMap = this.infor.find((inforItem) => {
            return item[inforItem.name]
              ?.toUpperCase()
              .includes(this.textSearch.toUpperCase());
          });
          if (isMap) {
            return true;
          } else {
            return false;
          }
        });
      }
    },
    //X??? l?? show, ch???n c??c option b???ng b??n ph??m
    handleKeyDownInput(e) {
      const allElementShowing = this.$refs.listCbb.querySelectorAll("tr");
      switch (e.keyCode) {
        case 9:
          this.$emit("handleTab");
          break;

        case 40:
          //Khi ??ang show list item
          if (this.isShowList) {
            if (allElementShowing.length > 0) {
              const currElement = this.$refs.listCbb.querySelector("tr.active");
              //N???u kh??ng c?? item n??o ??ang ???????c ch???n => focus item ?????u ti??n
              if (!currElement) {
                this.$refs.listCbb
                  .querySelector("tr.cbbItem-0")
                  .classList.add("active");
              } else if (
                //N???u ??ang focus v??o item cu???i => focus item ?????u
                currElement.classList.contains(
                  `cbbItem-${allElementShowing.length - 1}`
                )
              ) {
                currElement.classList.remove("active");
                this.$refs.listCbb
                  .querySelector("tr.cbbItem-0")
                  .classList.add("active");
              } else {
                //else => focus v??o item ti???p theo
                currElement.classList.remove("active");
                currElement.nextElementSibling.classList.add("active");
              }
            }
          }
          break;
        case 38:
          //Khi ??ang show list item
          if (this.isShowList) {
            if (allElementShowing.length > 0) {
              const currElement = this.$refs.listCbb.querySelector("tr.active");
              if (!currElement) {
                this.$refs.listCbb
                  .querySelector(`tr.cbbItem-${allElementShowing.length - 1}`)
                  .classList.add("active");
              } else if (currElement.classList.contains(`cbbItem-0`)) {
                currElement.classList.remove("active");
                this.$refs.listCbb
                  .querySelector(`tr.cbbItem-${allElementShowing.length - 1}`)
                  .classList.add("active");
              } else {
                currElement.classList.remove("active");
                currElement.previousElementSibling.classList.add("active");
              }
            }
          }
          break;
        case 13:
          if (!this.isShowList) {
            const coordinates = this.$refs.input_search.getBoundingClientRect();
            const inputHeight = this.$refs.input_search.offsetHeight;
            this.inputWidth = this.$refs.input_search.offsetWidth;
            this.dropDownX = coordinates.x;
            if (coordinates.y > 400) {
              this.dropDownY = coordinates.y - 200;
            } else {
              this.dropDownY = coordinates.y + inputHeight;
            }
            this.isShowList = true;
            this.dataChanged = this.data;
          } else {
            const currElement = this.$refs.listCbb.querySelector("tr.active");
            if (currElement) {
              currElement.click();
            }

            this.isShowList = false;
          }

          break;
        default:
          break;
      }
    },
  },
};
</script>

<style lang="scss">
@import "../../css/common/BaseComboboxTB.scss";
.m-input-error
{
  border-color: red !important;
}
</style>
