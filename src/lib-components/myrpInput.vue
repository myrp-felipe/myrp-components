<template lang="pug">
.input-field
  i.material-icons.prefix(v-on:click="focar()", v-if="view.icone") {{ view.icone }}
  input(:disabled="disabled", :ref="name" @keypress.13="enter", @keydown.tab="tab", :type="type", :maxlength="maxlength", :minlength="minlength", :name="name", :value="modelValue",  @input="updateValue", @blur="validar", :id="view.id", :class="{ invalid: view.erro }", :required="required")
  label(v-if="!tooltip", :for="view.id", :class="{ required: required }") {{ label }}
  label.tooltipped(v-if="tooltip", :for="view.id", :class="{ required: required }", data-html="true", data-position="top", :data-tooltip="tooltip") {{ label }}
    i.material-icons._mlxxs info_outline
  span.invalid(v-if="view.erro") {{ view.erro }}
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue'

interface IDadosComponente {
  id: string,
  erro: string | null,
  elemento: any,
  icone: any,
  regras: any
}

export default defineComponent({
  props: {
    disabled: {
      type: Boolean,
      default: false
    },
    id: String,
    label: String,
    name: String,
    apenasNumeros: {
      type: Boolean,
      default: false
    },
    regra: String,
    modelValue: [String, Number],
    required: Boolean,
    icon: String,
    inputmode: String,
    type: {
      type: String,
      default: "text"
    },
    maxlength: String,
    minlength: String,
    tooltip: String
  },
  setup(props, context) {
    const view = ref<IDadosComponente>({
      id: '',
      erro: null,
      elemento: null,
      icone: null,
      regras: {
        email: {
          erro: "E-mail inválido",
          validar: "^[a-z0-9._]+\\@[a-z0-9._]+"
        },
        url: {
          erro: "URL inválida",
          validar: "^(http|https)+\\:\\/\\/[a-z0-9._]+"
        },
        "cnpj-cpf": {
          erro: "CNPJ ou CPF inválido",
          validar: "^\\d{11}$|^\\d{14}$",
          formatar: "99999999999999"
        },
        cnpj: {
          erro: "CNPJ inválido",
          validar: "^\\d{2}\\.\\d{3}\\.\\d{3}\\/\\d{4}\\-\\d{2}$",
          formatar: "99.999.999/9999-99"
        },
        cpf: {
          erro: "CPF inválido",
          validar: "^\\d{3}\\.\\d{3}\\.\\d{3}\\-\\d{2}$",
          formatar: "999.999.999-99"
        },
        fone: {
          erro: "Telefone inválido",
          validar: "^\\(\\d{2}\\) 9\\d{8}|\\(\\d{2}\\) \\d{8}$",
          formatar: "(99) 999999999"
        },
        cep: {
          erro: "CEP inválido",
          validar: "^\\d{5}\\-\\d{3}$",
          formatar: "99999-999"
        },
        data: {
          erro: "Data inválida",
          validar: "^\\d{2}\\/\\d{2}\\/\\d{4}$",
          formatar: "99/99/9999"
        },
        "data-hora": {
          erro: "Data e hora inválida",
          validar: "^\\d{2}\\/\\d{2}\\/\\d{4} \\d{2}\\:\\d{2}$",
          formatar: "99/99/9999 99:99"
        },
        hora: {
          erro: "Hora inválida",
          validar: "^\\d{2}\\:\\d{2}$",
          formatar: "99:99"
        },
        ncm: {
          erro: "NCM inválido",
          validar: "^(\\d{2}|\\d{8})$",
          formatar: "99999999"
        },
        cest: {
          erro: "CEST inválido",
          validar: "^(\\d{7})$",
          formatar: "9999999"
        },
        ean: {
          erro: "GTIN/EAN inválido",
          validar: "^([0-9]{8}|[0-9]{12,14})$",
          formatar: "99999999999999"
        },
        "cartao-numero": {
          erro: "Número do cartão inválido",
          validar: "^\\d{4} \\d{4} \\d{4} \\d{4}",
          formatar: "9999 9999 9999 9999"
        },
        "cartao-codigo-seguranca": {
          erro: "Código de segurança inválido",
          validar: "^(\\d{3})$",
          formatar: "999"
        },
        placa: {
          erro: "Placa inválida",
          validar: "^[A-Za-z]{3}[0-9][A-Za-z0-9][0-9]{2}$",
          formatar: "zzz9*99"
        },
        "tamanho-minimo": {
          erro: `Informe ${props.minlength} ou mais caracteres`,
          validar: `^\\w[\\w\\s]{${parseInt(props.minlength || '') - 1}}`
        },
      }
    })

    function tab(valor: any) {
      context.emit("pressionouTab", valor);
    };
    function enter(valor: any) {
      context.emit("pressionouEnter", valor);
    };
    function validar(event: any) {
      let valor = event.target.value;
      return realizarValidacao(valor);
    };
    function realizarValidacao(valor: string) {
      valor = valor.trim();
      view.value.erro = null;

      if (!view.value.elemento.is(":visible")) {
        return true;
      }

      if (view.value.elemento.is("[required]") && !valor) {
        view.value.erro = "Campo obrigatório";
        return false;
      }

      if (valor && props.regra) {
        var regra = view.value.regras[props.regra];
        if (regra) {
          var r = new RegExp(regra.validar);
          if (!r.test(valor)) {
            view.value.erro = regra.erro;
            return false;
          }
        }
      }

      context.emit("blur", view.value)
      return true;
    };
    function formatar(valor: String) {
      if (valor === null || valor === undefined) return valor;

      //TO-DO: VALIDAÇÃO A SER VERIFICADA
      if(props.apenasNumeros){
        valor = valor.replace(/[^0-9,]/g, '').replace(/(\,,*?)\,,*/g, '$1');
      }

      var regra = props.regra ? view.value.regras[props.regra] : null;
      if (!regra) return valor;

      if (!regra.formatar) return valor;


      var mascara = regra.formatar;
      var valorFormatado = valor.toString().replace(/[^\w]/gi, "");

      for (var i = 0; i < valorFormatado.length; i++) {
        if (mascara[i] == null) {
          valorFormatado = valorFormatado.substring(0, mascara.length);
          break;
        }

        if (mascara[i] == "*" && /[A-Za-z0-9]/.test(valorFormatado[i]))
          continue;

        if (mascara[i] == "9" && /[0-9]/.test(valorFormatado[i])) continue;

        if (mascara[i] == "z" && /[A-Za-z]/.test(valorFormatado[i])) continue;

        if (mascara[i] != "9" && mascara[i] != "z") {
          valorFormatado =
            valorFormatado.substring(0, i) +
            mascara[i] +
            valorFormatado.substring(i);
          continue;
        }

        valorFormatado.substring(0, i) + valorFormatado.substring(i + 1);
      }

      return valorFormatado;
    };
    function ehValido() {
      return view.value.erro != null;
    };
    function sincronizar(valor: any) {
      var valorFormatado = formatar(valor);
      view.value.elemento.val(valorFormatado);
    };
    function focar() {
      $(view.value.elemento).focus();
    };
    function updateValue(event:any) {
      sincronizar(event.target.value);
      context.emit('update:modelValue', event.target.value);
    }

    return {
      view,
      updateValue,
      tab,
      enter,
      validar,
      realizarValidacao,
      formatar,
      ehValido,
      sincronizar,
      focar
    }
  },
  beforeMount: function () {
    this.view.id = this.id ? this.id : myrp.utils.id();
    this.view.icone = this.icon;
  },
  mounted: function () {
    this.view.elemento = $("#" + this.view.id);
    this.sincronizar(this.view.elemento.val());

    if (this.tooltip) {
      const element: any = $("[for=" + this.view.id + "]");
      if (element) {
        element.tooltip();
      }
    }

    Vue.nextTick(function () {
      Materialize.updateTextFields();
    });
  },
})
</script>

<style scoped>
</style>