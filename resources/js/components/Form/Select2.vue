<template>
  <div class="form-group mb-3" :class="dataError ? 'has-error' : validClass">
    <label for="#" class="text-muted" :class="prepend ? ' d-none': ''">{{ label }}</label>
    <select :name="name" class="form-control select2" :class="addClass" :multiple="multiple" selected="value">
      <slot></slot>
    </select>
    <div v-if="dataError" class="text-danger text-sm">
      {{ dataErrorMessage }}
    </div>
    <small v-if="info" class="form-text text-muted">
      {{ info }}
    </small>
  </div>
</template>

<script>
export default {
  name: "Select2",

  props: {
    addClass: {
      type: String,
      value: ''
    },
    options: {
      type: Array,
      value: [],
    },
    name: null,
    info : {
      type: String,
      value: ''
    },
    placeholder: {
      type: String,
      value: "",
    },
    label: {
      type: String,
      value: "",
    },
    multiple: {
      type: Boolean,
      value: false,
    },
    defaultValue: {
      type: String,
      value: ''
    },
    error: {
      type: Boolean,
      value: false
    },
    errorMessage: {
      type: String,
      value: null
    },
    url: '',
    keytext: '',
    text: '',
    old: null,
    prepend: false,
    getValue: false,

  },

  data() {
    return {
      dataError: false,
      dataErrorMessage: '',
      validClass: '',
      value: '',
    }
  },

  methods: {
    ajaxOptions() {
      let keytext = this.keytext
      let text = this.text
      return {
          url: this.url,
          dataType: 'Json',
          delay: 250,
          tags: true,
          data: function(params) {
            return {
              key: text,
              term: params.term,
              type: 'select2'
            }
          },
          processResults: function(data) {
            let res = data.payload.map((el) => {
              return {
                id: el[keytext],
                text: el[text]
              }
            })
            return {
              results: res
            }
          }
        }
    },
    getOptions(data) {
      let option;
      if (data.payload.length > 1) {
        let arr = [];
        for (let i = 0, len = data.payload.length; i < len; i++) {
          arr[i] = new Option(data.payload[i][this.text], data.payload[i][this.keytext], true, true);
        }
        option = arr
      } else {
        option = new Option(data.payload[0][this.text], data.payload[0][this.keytext], true, true);
      }

      return option;
    }
  },

  async mounted() {
    let vm = this;
    let selectElement = this.$el.children[1];
    let option;
    let property = {
      width: '100%',
      placeholder: this.placeholder,
    }
    if (this.url) {
      property.ajax = this.ajaxOptions()
      if (this.old !== "null") {
        let { data } = await axios.get(`${this.url}?type=select2&oldValue=${JSON.parse(this.old)}&key=${this.text}`)
        option = this.getOptions(data)
      }
      if (this.defaultValue) {
        let { data } = await axios.get(`${this.url}?type=select2&oldValue=${this.defaultValue}&key=${this.text}`)
        option = this.getOptions(data)
      }
    } else {
      property.data = this.options
      if (this.error) {
        this.dataErrorMessage = this.errorMessage
        this.dataError = this.error
      }
      if (this.defaultValue) {
        let defaultValue = ''
        if (!this.multiple) {
          defaultValue = this.defaultValue
        } else {
          defaultValue = JSON.parse(this.defaultValue)
        }
        this.value = defaultValue
      }
      if (this.old != 'null') {
        this.value = JSON.parse(this.old)
      }
    }
    $(selectElement)
      // init select2
      .select2(property)
      .val(this.value)
      .append(option)
      .trigger("change")
      // emit event on change.
      .on("change", function () {
        if (vm.getValue) {
          vm.$parent.receiveValue(this.value)
        }
        vm.$emit("input", this.value);
      });
  },
  watch: {
    value: function (value) {
      // update value
      $(this.$el).val(value).trigger("change");
      // this.ajaxOptions().url = this.url
      // $(this.$el).select2({ ajax: this.ajaxOptions() })
    },
    options: function (options) {
      // update options
      $(this.$el).empty().select2({ data: options });
    },
  },
  destroyed: function () {
    $(this.$el).off().select2("destroy");
  },
};
</script>

<style>
.has-error .select2-selection {
  border-color: rgb(185, 74, 72) !important;
}
</style>
