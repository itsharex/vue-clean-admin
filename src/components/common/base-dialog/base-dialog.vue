<script setup lang="ts">
import { computed, ref, useAttrs } from 'vue';
import { type DialogProps, ElDialog } from 'element-plus';
import { BaseDialogEmits, BaseDialogProps } from './typing';
import { DialogInstance } from 'element-plus';
import { omit } from 'lodash-es';
import type { ButtonProps } from 'element-plus';

defineOptions({
  name: 'BaseDialog',
});

const props = withDefaults(defineProps<BaseDialogProps>(), {
  title: '标题',
  showFooter: true,
  showCancelBtn: true,
  showConfirmBtn: true,
  showCloseIcon: true,
  cancelBtnText: '取消',
  confirmBtnText: '确定',
});

const emits = defineEmits<BaseDialogEmits>();

const dialogVisible = defineModel<boolean>({
  default: false,
});

/**
 * 是否存在 ActionBtns
 */
const isActionBtns = computed(() => {
  return props.actionBtns && Array.isArray(props.actionBtns) && props.actionBtns.length;
});

const dialogInstance = ref<NullType<DialogInstance>>();

/**
 * 获取 el-dialog 实例
 */
const getDialogInstance = computed(() => {
  return dialogInstance.value;
});

/**
 * el-dialog 属性
 */
const dialogAttrs = computed(() => {
  const attrs = useAttrs();

  // 要省略的属性
  const ignoreProps: Array<keyof DialogProps> = ['showClose', 'title'];

  const elDialogAttrs = omit(attrs, ignoreProps);

  const defaultAttrs: Partial<DialogProps> = {
    showClose: false,
  };

  return Object.assign(elDialogAttrs, defaultAttrs);
});

/**
 * 取消按钮属性
 */
const cancelBtnAttrs = computed(() => {
  // 默认属性
  const defaultAttrs: Partial<ButtonProps> = {
    plain: true,
  };
  return Object.assign(defaultAttrs, props.cancelBtnProps);
});

/**
 * 确认按钮属性
 */
const confirmBtnAttrs = computed(() => {
  // 默认属性
  const defaultAttrs: Partial<ButtonProps> = {
    type: 'primary',
  };
  return Object.assign(defaultAttrs, props.confirmBtnProps);
});

/**
 * 确认按钮点击事件
 */
const handleConfirm = () => {
  emits('handleConfirm');
};

/**
 * 取消按钮点击事件
 */
const handleCancel = () => {
  emits('handleCancel');
};

defineExpose({
  getDialogInstance,
});
</script>

<template>
  <ElDialog
    ref="dialogInstance"
    v-model="dialogVisible"
    v-bind="dialogAttrs"
    :class="['base-dialog', dialogClass]"
    @open="emits('open')"
    @opened="emits('opened')"
    @close="emits('close')"
    @closed="emits('closed')"
    @open-auto-focus="emits('openAutoFocus')"
    @close-auto-focus="emits('closeAutoFocus')"
  >
    <div v-if="showCloseIcon" class="absolute right-2 top-3">
      <ActionButton icon="mingcute:close-fill" :bg="false" @click="dialogVisible = false" />
    </div>

    <!-- 头部插槽 -->
    <template v-if="$slots.header" #header>
      <slot name="header" />
    </template>
    <template v-else #header>
      <div class="pr-4" :class="headerClass">
        <span class="text-lg font-medium">{{ title }}</span>
      </div>
    </template>

    <!-- 内容插槽 -->
    <slot />

    <!-- 底部插槽 -->
    <template v-if="showFooter" #footer>
      <slot v-if="$slots.footer" name="footer" />
      <div v-else-if="isActionBtns">
        <template v-for="(btn, index) in actionBtns" :key="index">
          <ElButton v-bind="omit(btn, 'btnText')" @click="btn.onClick">
            {{ btn.btnText }}
          </ElButton>
        </template>
      </div>

      <div v-else class="dialog-footer flex items-center justify-end" :class="footerClass">
        <!-- footer前置插槽 -->
        <slot name="footer-before" />
        <ElButton v-if="showCancelBtn" v-bind="cancelBtnAttrs" @click="handleCancel">
          {{ cancelBtnText }}
        </ElButton>
        <ElButton v-if="showConfirmBtn" v-bind="confirmBtnAttrs" @click="handleConfirm">
          {{ confirmBtnText }}
        </ElButton>
        <!-- footer后置插槽 -->
        <slot name="footer-after" />
      </div>
    </template>
  </ElDialog>
</template>

<style lang="scss">
@use './style.scss' as *;
</style>
