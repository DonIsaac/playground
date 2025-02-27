<script lang="ts">
import {
  type PropType,
  defineComponent,
  onBeforeUnmount,
  onMounted,
  ref,
} from "vue";
import { useDark } from "@vueuse/core";
import * as monaco from "monaco-editor";
import "src/composable/useEditorWorker";

export default defineComponent({
  name: "MonacoEditor",
  props: {
    width: {
      type: [String, Number],
      default: "100%",
    },
    height: {
      type: [String, Number],
      default: "100%",
    },
    original: String,
    value: String,
    language: { type: String },
    theme: { type: String },
    options: {
      type: Object as PropType<monaco.editor.IStandaloneEditorConstructionOptions>,
      default: () => ({}),
    },
  },
  emits: ["editorWillMount", "editorDidMount", "change", "update:modelValue"],
  setup(props, { emit }) {
    const container: any = ref(null);
    let instance: monaco.editor.IStandaloneCodeEditor | any = null;

    const isDark = useDark({
      onChanged(isDark) {
        if (!instance) return;
        monaco.editor.setTheme(isDark ? "vs-dark" : "vs");
      },
    });

    const initMonaco = () => {
      const editorProps = {
        minimap: { enabled: false },
        fontSize: 14,
        scrollBeyondLastLine: true,
        fontFamily: `ui-monospace, Menlo, Monaco, "Cascadia Code", "Cascadia Mono", "Segoe UI Mono", "Roboto Mono", "Oxygen Mono", "Ubuntu Monospace", "Source Code Pro","Fira Mono", "Droid Sans Mono", "Courier New", monospace`,
        locale: "en",
        ...props,
        theme: props.theme || (isDark.value ? "vs-dark" : "vs"),
      };

      instance = monaco.editor.create(container.value, editorProps);
      instance.onDidChangeModelContent(() => {
        const value = instance.getValue();
        emit("update:modelValue", value);
      });
    };

    onMounted(() => {
      initMonaco();
    });

    onBeforeUnmount(() => {
      if (instance) {
        instance.dispose();
      }
    });

    return {
      container,
    };
  },
});
</script>

<template>
  <div class="w-full h-full" ref="container" />
</template>
