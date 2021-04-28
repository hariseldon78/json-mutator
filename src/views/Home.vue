<template>
    <div>
        <v-textarea label="input json" v-model="input"></v-textarea>
        <v-btn @click="input=JSON.stringify(JSON.parse(input),null,4)">Pretty!</v-btn>

        <p>Combinations: {{combinations.length}}</p>
        <v-container>
            <div v-for="(field, findex) in fields" :key="findex">
                <v-row>
                    <v-text-field
                        label="field to mutate"
                        v-model="field.path"
                    ></v-text-field>
                    <template v-for="(value, index) in field.values">
                        <v-col :key="index" >
                            <v-text-field
                                label="value"
                                v-model="fields[findex].values[index]"
                            ></v-text-field>
                        </v-col>
                    </template>
                    <v-btn @click="fields[findex].values.push('')">add value</v-btn>
                </v-row>
            </div>

            <v-btn @click="fields.push({ path: '', values: [0] })">add field</v-btn>
        </v-container>
        <v-btn @click="mutate">Mutate!</v-btn>
        <v-textarea label="output" v-model="output"></v-textarea>
    </div>
    
</template>

<script lang="ts">
import Vue from 'vue';
import { Component } from 'vue-property-decorator';
import * as _ from 'lodash';
interface Field {
    path: string;
    values: number[];
}
@Component({})
export default class Home extends Vue {
    input = '';
    output='';
    fields: Field[] = [];
    mutate(){
        const obj=JSON.parse(this.input);
        const combs=this.combinations;
        this.output=JSON.stringify(this.combinations,null,4);
    }
    get combinations(){
        let res=[];
        for (const field of this.fields){
            if (!res.length){
                res.push(...field.values.map(v=>{const obj={};obj[field.path]=v;return obj;}))
            }
            else {
                const newRes=[];
                for (const value of field.values){
                    for (const comb of res){
                        const clone={...comb};
                        clone[field.path]=value;
                        newRes.push(clone);
                    }
                }
                res=[...newRes];
            }
        }
        console.log(res);
        return res;
    }
}
</script>
