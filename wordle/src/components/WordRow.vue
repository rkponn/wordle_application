<script setup>
import { watch, ref } from 'vue';
import LetterBox from './LetterBox.vue';
const props = defineProps({
    value: String,
    solution: String,
    submitted: Boolean,
});

const colors = ref(["","","","",""]);

watch(
    () => props.submitted,
    async (submitted, previouslySubmitted) => {
        if(props.submitted) {
            let solution = props.solution;
            let val = props.value;

            let boxColor = ["gray", "gray", "gray", "gray", "gray"];
            let letterStack = [];

            for (let i = 0; i < 5; i++) {
                if (solution.charAt(i) == val.charAt(i)) {
                    boxColor[i] = "green";
                } else {
                    letterStack.push(solution.charAt(i));
                }
            }

            for (let i = 0; i < 5; i++) {
                if (boxColor[i] == "gray") {
                    if (letterStack.indexOf(val.charAt(i)) != -1) {
                        letterStack.splice(letterStack.indexOf(val.charAt(i)), 1);
                        boxColor[i] = "yellow";
                    }
                }
                colors.value[i] = boxColor[i];
                await new Promise((resolve) => setTimeout(resolve, 50));
            }
        }
    }
);
</script>

<template>
    <div class="grid max-w-xs grid-cols-5 gap-1 mx-auto mb-1">
        <LetterBox
            v-for="i in 5"
            :key="i"
            :letter="value[i-1]"
            :color="colors[i-1]" />
    </div>
</template>
<style></style>