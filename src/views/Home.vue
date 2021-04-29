<template>
    <div>
        <v-textarea label="input json" v-model="input"></v-textarea>
        <v-btn @click="input = JSON.stringify(JSON.parse(input), null, 4)">Pretty!</v-btn>

        <p>Combinations: {{ combinations.length }}</p>
        <v-container>
            <div v-for="(field, findex) in idFields" :key="findex">
                <v-row >
                    <v-text-field label="id field" v-model="field.path"></v-text-field>
                    <v-col>
                        <v-text-field
                            label="start from"
                            v-model="field.startFrom"
                        ></v-text-field>
                    </v-col>
                    <v-btn @click="idFields.splice(findex,1)">remove</v-btn>
                </v-row>
            </div>
            <div v-for="(field, findex) in fields" :key="findex">
                <v-row>
                    <v-text-field
                        label="field to mutate"
                        v-model="field.path"
                    ></v-text-field>
                    <template v-for="(value, index) in field.values">
                        <v-col :key="index">
                            <v-text-field
                                label="value"
                                v-model="fields[findex].values[index]"
                            ></v-text-field>
                        </v-col>
                    </template>
                    <v-btn @click="fields[findex].values.push('')">add value</v-btn>
                    <v-btn @click="fields.splice(findex,1)">remove</v-btn>
                </v-row>
            </div>

            <v-btn @click="fields.push({ path: '', values: [0] })">
                add mutating field
            </v-btn>
            <v-btn @click="idFields.push({ path: '',startFrom:0 })">add id field</v-btn>
        </v-container>
        <v-btn @click="mutate">Mutate!</v-btn>
        <v-textarea label="output" v-model="output"></v-textarea>
    </div>
</template>

<script lang="ts">
import Vue from 'vue';
import { Component } from 'vue-property-decorator';
import * as _ from 'lodash';
interface MutatingField {
    path: string;
    values: number[];
}
interface IdField {
    path: string;
    startFrom: number;
}
interface CurrentIdField {
    path: string;
    current: number;
}
type FieldValue = number | string;
@Component({})
export default class Home extends Vue {
    input = '';
    output = '';
    fields: MutatingField[] = [];
    idFields: IdField[] = [];
    mutate() {
        const obj = JSON.parse(this.input);
        const combs = this.combinations;
        const res = [];
        const ids: CurrentIdField[] = this.idFields.map((f) => ({
            path: f.path,
            current: f.startFrom,
        }));
        for (const comb of combs) {
            const mutated = _.cloneDeep(obj);
            Object.keys(comb).forEach((f) => _.set(mutated, f, comb[f]));
            ids.forEach((idField) => {
                _.set(mutated, idField.path, ''+idField.current);
                idField.current++;
            });
            res.push(mutated);
        }
        this.output = JSON.stringify(res, null, 4);
    }
    get combinations(): { [f: string]: FieldValue }[] {
        let res: { [f: string]: any }[] = [];
        for (const field of this.fields) {
            if (!res.length) {
                res.push(
                    ...field.values.map((v: FieldValue) => {
                        const obj: { [f: string]: any } = {};
                        obj[field.path] = v;
                        return obj;
                    })
                );
            } else {
                const newRes = [];
                for (const value of field.values) {
                    for (const comb of res) {
                        const clone = { ...comb };
                        clone[field.path] = value;
                        newRes.push(clone);
                    }
                }
                res = [...newRes];
            }
        }
        console.log(res);
        return res;
    }
}
</script>
