<script setup lang="ts">
import {router, useForm} from '@inertiajs/vue3';
import {onMounted, ref} from "vue";

const props = defineProps<{
    schema: any;
    resetOnSuccess?: boolean;
}>();

const vueForm = ref<any>(null);

const fillValues = () => {
    const initialValues: any = {};
    Object.entries(props.schema.schema).forEach(([key, field]: Array<any>) => initialValues[key] = field.value);
    if (vueForm.value) {
        vueForm.value.update(initialValues);
    }
};
let finish = router.on('finish', () => {
    fillValues();
});
onMounted(() => {
    fillValues();
    finish();
});

const submitForm = async (FormData: FormData, form$: any) => {
    let data: any = {};
    FormData.forEach((value, key) => {
        data[key] = value;
    });

    const inertiaForm = useForm(data);
    inertiaForm.submit(props.schema.method.toLocaleLowerCase(), props.schema.endpoint, {
        preserveScroll: true,
        onError: (errors: object) => {
            Object.entries(errors).forEach(([key, value]) => {
                form$.el$(key).messageBag.prepend(value);
            });
            console.log('error');
        },
        onSuccess: () => {
            console.log('success');
            Object.entries(form$.elements$).forEach(([key, field]: Array<any>) => {
                if (field.messageBag) {
                    field.messageBag.clear();
                }
            });
            if (props.resetOnSuccess) {
                form$.clear();
                form$.reset();
            }
            form$.messageBag.append('Password successfully changed', 'success')
        }
    });
};

</script>

<template>
    <Vueform ref="vueForm" :endpoint="submitForm" :schema="schema.schema" :display-errors="false"/>
</template>
