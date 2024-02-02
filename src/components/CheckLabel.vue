<template>
    <div class="check-label">
        <input :id="id" type="checkbox" :value="modelValue" @change="change"/>
        <label :for="id">{{label}}</label>
    </div>
</template>

<script>
export default {
    name: "CheckLabel",
    props: {
        id: {
            type: String,
            required: true
        },
        modelValue: {
            type: Boolean,
            required: true
        },
        label: {
            type: String,
            required: true
        }
    },
    emits: ['update:modelValue'],
    setup(_, context) {
        const change = (ev) => {
            console.log(ev.target.value)
            context.emit('update:modelValue', ev.target.value)
        };
        return {
            change
        };
    }
};
</script>

<style lang="scss" scoped>
.check-label {
    display: flex;
    gap: 5px;
}
</style>
