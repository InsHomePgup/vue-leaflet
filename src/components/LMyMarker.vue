<script lang="ts">
import type L from "leaflet";
import { debounce } from "ts-debounce";
import {
  defineComponent,
  inject,
  markRaw,
  nextTick,
  onBeforeUnmount,
  onMounted,
  provide,
  ref,
} from "vue";

import { render } from "@src/functions/layer";
import {
  markerProps,
  setupMarker,
  shouldBlankIcon,
} from "@src/functions/marker";
import {
  AddLayerInjection,
  CanSetParentHtmlInjection,
  SetIconInjection,
  SetParentHtmlInjection,
  UseGlobalLeafletInjection,
} from "@src/types/injectionKeys";
import {
  WINDOW_OR_GLOBAL,
  assertInject,
  cancelDebounces,
  isFunction,
  propsBinder,
  remapEvents,
} from "@src/utils.js";

/**
 * Marker component, lets you add and personalize markers on the map
 */
export default defineComponent({
  name: "LMyMarker", // 注册组件name
  props: markerProps, // 注册组件props
  setup(props, context) {
    const leafletObject = ref<L.Marker>(); // 创建leaflet 对象
    const ready = ref(false); // 渲染完成标记

    const useGlobalLeaflet = inject(UseGlobalLeafletInjection); // 是否使用window下面的Leaflet
    const addLayer = assertInject(AddLayerInjection); // 安全注入

    /**
     * 提供三个HTML元素
     */
    provide(
      CanSetParentHtmlInjection,
      () => !!leafletObject.value?.getElement()
    );
    provide(SetParentHtmlInjection, (html: string) => {
      const el =
        isFunction(leafletObject.value?.getElement) &&
        leafletObject.value?.getElement();
      if (!el) return;
      el.innerHTML = html;
    });
    provide(
      SetIconInjection,
      (newIcon: L.DivIcon | L.Icon) =>
        leafletObject.value?.setIcon && leafletObject.value.setIcon(newIcon)
    );

    const { options, methods } = setupMarker(props, leafletObject, context);

    const eventHandlers = {
      moveHandler: debounce(methods.latLngSync),
    };

    onMounted(async () => {
      console.log("注入的内容=====", useGlobalLeaflet);

      const { marker, divIcon }: typeof L = useGlobalLeaflet
        ? WINDOW_OR_GLOBAL.L
        : await import("leaflet/dist/leaflet-src.esm");

      if (shouldBlankIcon(options, context)) {
        options.icon = divIcon({ className: "" });
      }
      leafletObject.value = markRaw<L.Marker>(marker(props.latLng, options));

      const { listeners } = remapEvents(context.attrs);
      leafletObject.value.on(listeners);

      leafletObject.value.on("move", eventHandlers.moveHandler);
      propsBinder(methods, leafletObject.value, props);
      addLayer({
        ...props,
        ...methods,
        leafletObject: leafletObject.value,
      });
      ready.value = true;
      nextTick(() => context.emit("ready", leafletObject.value));
    });

    onBeforeUnmount(() => cancelDebounces(eventHandlers));

    console.log("leaflet marker注册=====", ready, leafletObject);

    return { ready, leafletObject };
  },
  render() {
    return render(this.ready, this.$slots);
  },
});
</script>
