<template>
  <div class="h-screen w-screen">
    <div ref="threejsMap"></div>
  </div>
</template>

<script setup lang="ts">
import * as THREE from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { onMounted, ref } from 'vue';
import * as TWEEN from '@tweenjs/tween.js';
import { createBatchedMesh, createCube, createPlane } from '../utils/geometryGenerator.ts'


// const graphData = {
//   "nodes": [
//     { "id": "Myriel", "group": 1 },
//     { "id": "Napoleon", "group": 1 },
//     { "id": "Mlle.Baptistine", "group": 1 },
//     { "id": "Mme.Magloire", "group": 1 },
//     { "id": "CountessdeLo", "group": 1 },
//     { "id": "Geborand", "group": 1 },
//     { "id": "Champtercier", "group": 1 },
//     { "id": "Cravatte", "group": 1 },
//     { "id": "Count", "group": 1 },
//     { "id": "OldMan", "group": 1 },
//     { "id": "Labarre", "group": 2 },
//     { "id": "Valjean", "group": 2 },
//     { "id": "Marguerite", "group": 3 },
//     { "id": "Mme.deR", "group": 2 },
//     { "id": "Isabeau", "group": 2 },
//     { "id": "Gervais", "group": 2 },
//     { "id": "Tholomyes", "group": 3 },
//     { "id": "Listolier", "group": 3 },
//     { "id": "Fameuil", "group": 3 },
//     { "id": "Blacheville", "group": 3 },
//     { "id": "Favourite", "group": 3 },
//     { "id": "Dahlia", "group": 3 },
//     { "id": "Zephine", "group": 3 },
//     { "id": "Fantine", "group": 3 },
//     { "id": "Mme.Thenardier", "group": 4 },
//     { "id": "Thenardier", "group": 4 },
//     { "id": "Cosette", "group": 5 },
//     { "id": "Javert", "group": 4 },
//     { "id": "Fauchelevent", "group": 0 },
//     { "id": "Bamatabois", "group": 2 },
//     { "id": "Perpetue", "group": 3 },
//     { "id": "Simplice", "group": 2 },
//     { "id": "Scaufflaire", "group": 2 },
//     { "id": "Woman1", "group": 2 },
//     { "id": "Judge", "group": 2 },
//     { "id": "Champmathieu", "group": 2 },
//     { "id": "Brevet", "group": 2 },
//     { "id": "Chenildieu", "group": 2 },
//     { "id": "Cochepaille", "group": 2 },
//     { "id": "Pontmercy", "group": 4 },
//     { "id": "Boulatruelle", "group": 6 },
//     { "id": "Eponine", "group": 4 },
//     { "id": "Anzelma", "group": 4 },
//     { "id": "Woman2", "group": 5 },
//     { "id": "MotherInnocent", "group": 0 },
//     { "id": "Gribier", "group": 0 },
//     { "id": "Jondrette", "group": 7 },
//     { "id": "Mme.Burgon", "group": 7 },
//     { "id": "Gavroche", "group": 8 },
//     { "id": "Gillenormand", "group": 5 },
//     { "id": "Magnon", "group": 5 },
//     { "id": "Mlle.Gillenormand", "group": 5 },
//     { "id": "Mme.Pontmercy", "group": 5 },
//     { "id": "Mlle.Vaubois", "group": 5 },
//     { "id": "Lt.Gillenormand", "group": 5 },
//     { "id": "Marius", "group": 8 },
//     { "id": "BaronessT", "group": 5 },
//     { "id": "Mabeuf", "group": 8 },
//     { "id": "Enjolras", "group": 8 },
//     { "id": "Combeferre", "group": 8 },
//     { "id": "Prouvaire", "group": 8 },
//     { "id": "Feuilly", "group": 8 },
//     { "id": "Courfeyrac", "group": 8 },
//     { "id": "Bahorel", "group": 8 },
//     { "id": "Bossuet", "group": 8 },
//     { "id": "Joly", "group": 8 },
//     { "id": "Grantaire", "group": 8 },
//     { "id": "MotherPlutarch", "group": 9 },
//     { "id": "Gueulemer", "group": 4 },
//     { "id": "Babet", "group": 4 },
//     { "id": "Claquesous", "group": 4 },
//     { "id": "Montparnasse", "group": 4 },
//     { "id": "Toussaint", "group": 5 },
//     { "id": "Child1", "group": 10 },
//     { "id": "Child2", "group": 10 },
//     { "id": "Brujon", "group": 4 },
//     { "id": "Mme.Hucheloup", "group": 8 }
//   ],
//   "links": [
//     { "source": "Napoleon", "target": "Myriel", "value": 1 },
//     { "source": "Mlle.Baptistine", "target": "Myriel", "value": 8 },
//     { "source": "Mme.Magloire", "target": "Myriel", "value": 10 },
//     { "source": "Mme.Magloire", "target": "Mlle.Baptistine", "value": 6 },
//     { "source": "CountessdeLo", "target": "Myriel", "value": 1 },
//     { "source": "Geborand", "target": "Myriel", "value": 1 },
//     { "source": "Champtercier", "target": "Myriel", "value": 1 },
//     { "source": "Cravatte", "target": "Myriel", "value": 1 },
//     { "source": "Count", "target": "Myriel", "value": 2 },
//     { "source": "OldMan", "target": "Myriel", "value": 1 },
//     { "source": "Valjean", "target": "Labarre", "value": 1 },
//     { "source": "Valjean", "target": "Mme.Magloire", "value": 3 },
//     { "source": "Valjean", "target": "Mlle.Baptistine", "value": 3 },
//     { "source": "Valjean", "target": "Myriel", "value": 5 },
//     { "source": "Marguerite", "target": "Valjean", "value": 1 },
//     { "source": "Mme.deR", "target": "Valjean", "value": 1 },
//     { "source": "Isabeau", "target": "Valjean", "value": 1 },
//     { "source": "Gervais", "target": "Valjean", "value": 1 },
//     { "source": "Listolier", "target": "Tholomyes", "value": 4 },
//     { "source": "Fameuil", "target": "Tholomyes", "value": 4 },
//     { "source": "Fameuil", "target": "Listolier", "value": 4 },
//     { "source": "Blacheville", "target": "Tholomyes", "value": 4 },
//     { "source": "Blacheville", "target": "Listolier", "value": 4 },
//     { "source": "Blacheville", "target": "Fameuil", "value": 4 },
//     { "source": "Favourite", "target": "Tholomyes", "value": 3 },
//     { "source": "Favourite", "target": "Listolier", "value": 3 },
//     { "source": "Favourite", "target": "Fameuil", "value": 3 },
//     { "source": "Favourite", "target": "Blacheville", "value": 4 },
//     { "source": "Dahlia", "target": "Tholomyes", "value": 3 },
//     { "source": "Dahlia", "target": "Listolier", "value": 3 },
//     { "source": "Dahlia", "target": "Fameuil", "value": 3 },
//     { "source": "Dahlia", "target": "Blacheville", "value": 3 },
//     { "source": "Dahlia", "target": "Favourite", "value": 5 },
//     { "source": "Zephine", "target": "Tholomyes", "value": 3 },
//     { "source": "Zephine", "target": "Listolier", "value": 3 },
//     { "source": "Zephine", "target": "Fameuil", "value": 3 },
//     { "source": "Zephine", "target": "Blacheville", "value": 3 },
//     { "source": "Zephine", "target": "Favourite", "value": 4 },
//     { "source": "Zephine", "target": "Dahlia", "value": 4 },
//     { "source": "Fantine", "target": "Tholomyes", "value": 3 },
//     { "source": "Fantine", "target": "Listolier", "value": 3 },
//     { "source": "Fantine", "target": "Fameuil", "value": 3 },
//     { "source": "Fantine", "target": "Blacheville", "value": 3 },
//     { "source": "Fantine", "target": "Favourite", "value": 4 },
//     { "source": "Fantine", "target": "Dahlia", "value": 4 },
//     { "source": "Fantine", "target": "Zephine", "value": 4 },
//     { "source": "Fantine", "target": "Marguerite", "value": 2 },
//     { "source": "Fantine", "target": "Valjean", "value": 9 },
//     { "source": "Mme.Thenardier", "target": "Fantine", "value": 2 },
//     { "source": "Mme.Thenardier", "target": "Valjean", "value": 7 },
//     { "source": "Thenardier", "target": "Mme.Thenardier", "value": 13 },
//     { "source": "Thenardier", "target": "Fantine", "value": 1 },
//     { "source": "Thenardier", "target": "Valjean", "value": 12 },
//     { "source": "Cosette", "target": "Mme.Thenardier", "value": 4 },
//     { "source": "Cosette", "target": "Valjean", "value": 31 },
//     { "source": "Cosette", "target": "Tholomyes", "value": 1 },
//     { "source": "Cosette", "target": "Thenardier", "value": 1 },
//     { "source": "Javert", "target": "Valjean", "value": 17 },
//     { "source": "Javert", "target": "Fantine", "value": 5 },
//     { "source": "Javert", "target": "Thenardier", "value": 5 },
//     { "source": "Javert", "target": "Mme.Thenardier", "value": 1 },
//     { "source": "Javert", "target": "Cosette", "value": 1 },
//     { "source": "Fauchelevent", "target": "Valjean", "value": 8 },
//     { "source": "Fauchelevent", "target": "Javert", "value": 1 },
//     { "source": "Bamatabois", "target": "Fantine", "value": 1 },
//     { "source": "Bamatabois", "target": "Javert", "value": 1 },
//     { "source": "Bamatabois", "target": "Valjean", "value": 2 },
//     { "source": "Perpetue", "target": "Fantine", "value": 1 },
//     { "source": "Simplice", "target": "Perpetue", "value": 2 },
//     { "source": "Simplice", "target": "Valjean", "value": 3 },
//     { "source": "Simplice", "target": "Fantine", "value": 2 },
//     { "source": "Simplice", "target": "Javert", "value": 1 },
//     { "source": "Scaufflaire", "target": "Valjean", "value": 1 },
//     { "source": "Woman1", "target": "Valjean", "value": 2 },
//     { "source": "Woman1", "target": "Javert", "value": 1 },
//     { "source": "Judge", "target": "Valjean", "value": 3 },
//     { "source": "Judge", "target": "Bamatabois", "value": 2 },
//     { "source": "Champmathieu", "target": "Valjean", "value": 3 },
//     { "source": "Champmathieu", "target": "Judge", "value": 3 },
//     { "source": "Champmathieu", "target": "Bamatabois", "value": 2 },
//     { "source": "Brevet", "target": "Judge", "value": 2 },
//     { "source": "Brevet", "target": "Champmathieu", "value": 2 },
//     { "source": "Brevet", "target": "Valjean", "value": 2 },
//     { "source": "Brevet", "target": "Bamatabois", "value": 1 },
//     { "source": "Chenildieu", "target": "Judge", "value": 2 },
//     { "source": "Chenildieu", "target": "Champmathieu", "value": 2 },
//     { "source": "Chenildieu", "target": "Brevet", "value": 2 },
//     { "source": "Chenildieu", "target": "Valjean", "value": 2 },
//     { "source": "Chenildieu", "target": "Bamatabois", "value": 1 },
//     { "source": "Cochepaille", "target": "Judge", "value": 2 },
//     { "source": "Cochepaille", "target": "Champmathieu", "value": 2 },
//     { "source": "Cochepaille", "target": "Brevet", "value": 2 },
//     { "source": "Cochepaille", "target": "Chenildieu", "value": 2 },
//     { "source": "Cochepaille", "target": "Valjean", "value": 2 },
//     { "source": "Cochepaille", "target": "Bamatabois", "value": 1 },
//     { "source": "Pontmercy", "target": "Thenardier", "value": 1 },
//     { "source": "Boulatruelle", "target": "Thenardier", "value": 1 },
//     { "source": "Eponine", "target": "Mme.Thenardier", "value": 2 },
//     { "source": "Eponine", "target": "Thenardier", "value": 3 },
//     { "source": "Anzelma", "target": "Eponine", "value": 2 },
//     { "source": "Anzelma", "target": "Thenardier", "value": 2 },
//     { "source": "Anzelma", "target": "Mme.Thenardier", "value": 1 },
//     { "source": "Woman2", "target": "Valjean", "value": 3 },
//     { "source": "Woman2", "target": "Cosette", "value": 1 },
//     { "source": "Woman2", "target": "Javert", "value": 1 },
//     { "source": "MotherInnocent", "target": "Fauchelevent", "value": 3 },
//     { "source": "MotherInnocent", "target": "Valjean", "value": 1 },
//     { "source": "Gribier", "target": "Fauchelevent", "value": 2 },
//     { "source": "Mme.Burgon", "target": "Jondrette", "value": 1 },
//     { "source": "Gavroche", "target": "Mme.Burgon", "value": 2 },
//     { "source": "Gavroche", "target": "Thenardier", "value": 1 },
//     { "source": "Gavroche", "target": "Javert", "value": 1 },
//     { "source": "Gavroche", "target": "Valjean", "value": 1 },
//     { "source": "Gillenormand", "target": "Cosette", "value": 3 },
//     { "source": "Gillenormand", "target": "Valjean", "value": 2 },
//     { "source": "Magnon", "target": "Gillenormand", "value": 1 },
//     { "source": "Magnon", "target": "Mme.Thenardier", "value": 1 },
//     { "source": "Mlle.Gillenormand", "target": "Gillenormand", "value": 9 },
//     { "source": "Mlle.Gillenormand", "target": "Cosette", "value": 2 },
//     { "source": "Mlle.Gillenormand", "target": "Valjean", "value": 2 },
//     { "source": "Mme.Pontmercy", "target": "Mlle.Gillenormand", "value": 1 },
//     { "source": "Mme.Pontmercy", "target": "Pontmercy", "value": 1 },
//     { "source": "Mlle.Vaubois", "target": "Mlle.Gillenormand", "value": 1 },
//     { "source": "Lt.Gillenormand", "target": "Mlle.Gillenormand", "value": 2 },
//     { "source": "Lt.Gillenormand", "target": "Gillenormand", "value": 1 },
//     { "source": "Lt.Gillenormand", "target": "Cosette", "value": 1 },
//     { "source": "Marius", "target": "Mlle.Gillenormand", "value": 6 },
//     { "source": "Marius", "target": "Gillenormand", "value": 12 },
//     { "source": "Marius", "target": "Pontmercy", "value": 1 },
//     { "source": "Marius", "target": "Lt.Gillenormand", "value": 1 },
//     { "source": "Marius", "target": "Cosette", "value": 21 },
//     { "source": "Marius", "target": "Valjean", "value": 19 },
//     { "source": "Marius", "target": "Tholomyes", "value": 1 },
//     { "source": "Marius", "target": "Thenardier", "value": 2 },
//     { "source": "Marius", "target": "Eponine", "value": 5 },
//     { "source": "Marius", "target": "Gavroche", "value": 4 },
//     { "source": "BaronessT", "target": "Gillenormand", "value": 1 },
//     { "source": "BaronessT", "target": "Marius", "value": 1 },
//     { "source": "Mabeuf", "target": "Marius", "value": 1 },
//     { "source": "Mabeuf", "target": "Eponine", "value": 1 },
//     { "source": "Mabeuf", "target": "Gavroche", "value": 1 },
//     { "source": "Enjolras", "target": "Marius", "value": 7 },
//     { "source": "Enjolras", "target": "Gavroche", "value": 7 },
//     { "source": "Enjolras", "target": "Javert", "value": 6 },
//     { "source": "Enjolras", "target": "Mabeuf", "value": 1 },
//     { "source": "Enjolras", "target": "Valjean", "value": 4 },
//     { "source": "Combeferre", "target": "Enjolras", "value": 15 },
//     { "source": "Combeferre", "target": "Marius", "value": 5 },
//     { "source": "Combeferre", "target": "Gavroche", "value": 6 },
//     { "source": "Combeferre", "target": "Mabeuf", "value": 2 },
//     { "source": "Prouvaire", "target": "Gavroche", "value": 1 },
//     { "source": "Prouvaire", "target": "Enjolras", "value": 4 },
//     { "source": "Prouvaire", "target": "Combeferre", "value": 2 },
//     { "source": "Feuilly", "target": "Gavroche", "value": 2 },
//     { "source": "Feuilly", "target": "Enjolras", "value": 6 },
//     { "source": "Feuilly", "target": "Prouvaire", "value": 2 },
//     { "source": "Feuilly", "target": "Combeferre", "value": 5 },
//     { "source": "Feuilly", "target": "Mabeuf", "value": 1 },
//     { "source": "Feuilly", "target": "Marius", "value": 1 },
//     { "source": "Courfeyrac", "target": "Marius", "value": 9 },
//     { "source": "Courfeyrac", "target": "Enjolras", "value": 17 },
//     { "source": "Courfeyrac", "target": "Combeferre", "value": 13 },
//     { "source": "Courfeyrac", "target": "Gavroche", "value": 7 },
//     { "source": "Courfeyrac", "target": "Mabeuf", "value": 2 },
//     { "source": "Courfeyrac", "target": "Eponine", "value": 1 },
//     { "source": "Courfeyrac", "target": "Feuilly", "value": 6 },
//     { "source": "Courfeyrac", "target": "Prouvaire", "value": 3 },
//     { "source": "Bahorel", "target": "Combeferre", "value": 5 },
//     { "source": "Bahorel", "target": "Gavroche", "value": 5 },
//     { "source": "Bahorel", "target": "Courfeyrac", "value": 6 },
//     { "source": "Bahorel", "target": "Mabeuf", "value": 2 },
//     { "source": "Bahorel", "target": "Enjolras", "value": 4 },
//     { "source": "Bahorel", "target": "Feuilly", "value": 3 },
//     { "source": "Bahorel", "target": "Prouvaire", "value": 2 },
//     { "source": "Bahorel", "target": "Marius", "value": 1 },
//     { "source": "Bossuet", "target": "Marius", "value": 5 },
//     { "source": "Bossuet", "target": "Courfeyrac", "value": 12 },
//     { "source": "Bossuet", "target": "Gavroche", "value": 5 },
//     { "source": "Bossuet", "target": "Bahorel", "value": 4 },
//     { "source": "Bossuet", "target": "Enjolras", "value": 10 },
//     { "source": "Bossuet", "target": "Feuilly", "value": 6 },
//     { "source": "Bossuet", "target": "Prouvaire", "value": 2 },
//     { "source": "Bossuet", "target": "Combeferre", "value": 9 },
//     { "source": "Bossuet", "target": "Mabeuf", "value": 1 },
//     { "source": "Bossuet", "target": "Valjean", "value": 1 },
//     { "source": "Joly", "target": "Bahorel", "value": 5 },
//     { "source": "Joly", "target": "Bossuet", "value": 7 },
//     { "source": "Joly", "target": "Gavroche", "value": 3 },
//     { "source": "Joly", "target": "Courfeyrac", "value": 5 },
//     { "source": "Joly", "target": "Enjolras", "value": 5 },
//     { "source": "Joly", "target": "Feuilly", "value": 5 },
//     { "source": "Joly", "target": "Prouvaire", "value": 2 },
//     { "source": "Joly", "target": "Combeferre", "value": 5 },
//     { "source": "Joly", "target": "Mabeuf", "value": 1 },
//     { "source": "Joly", "target": "Marius", "value": 2 },
//     { "source": "Grantaire", "target": "Bossuet", "value": 3 },
//     { "source": "Grantaire", "target": "Enjolras", "value": 3 },
//     { "source": "Grantaire", "target": "Combeferre", "value": 1 },
//     { "source": "Grantaire", "target": "Courfeyrac", "value": 2 },
//     { "source": "Grantaire", "target": "Joly", "value": 2 },
//     { "source": "Grantaire", "target": "Gavroche", "value": 1 },
//     { "source": "Grantaire", "target": "Bahorel", "value": 1 },
//     { "source": "Grantaire", "target": "Feuilly", "value": 1 },
//     { "source": "Grantaire", "target": "Prouvaire", "value": 1 },
//     { "source": "MotherPlutarch", "target": "Mabeuf", "value": 3 },
//     { "source": "Gueulemer", "target": "Thenardier", "value": 5 },
//     { "source": "Gueulemer", "target": "Valjean", "value": 1 },
//     { "source": "Gueulemer", "target": "Mme.Thenardier", "value": 1 },
//     { "source": "Gueulemer", "target": "Javert", "value": 1 },
//     { "source": "Gueulemer", "target": "Gavroche", "value": 1 },
//     { "source": "Gueulemer", "target": "Eponine", "value": 1 },
//     { "source": "Babet", "target": "Thenardier", "value": 6 },
//     { "source": "Babet", "target": "Gueulemer", "value": 6 },
//     { "source": "Babet", "target": "Valjean", "value": 1 },
//     { "source": "Babet", "target": "Mme.Thenardier", "value": 1 },
//     { "source": "Babet", "target": "Javert", "value": 2 },
//     { "source": "Babet", "target": "Gavroche", "value": 1 },
//     { "source": "Babet", "target": "Eponine", "value": 1 },
//     { "source": "Claquesous", "target": "Thenardier", "value": 4 },
//     { "source": "Claquesous", "target": "Babet", "value": 4 },
//     { "source": "Claquesous", "target": "Gueulemer", "value": 4 },
//     { "source": "Claquesous", "target": "Valjean", "value": 1 },
//     { "source": "Claquesous", "target": "Mme.Thenardier", "value": 1 },
//     { "source": "Claquesous", "target": "Javert", "value": 1 },
//     { "source": "Claquesous", "target": "Eponine", "value": 1 },
//     { "source": "Claquesous", "target": "Enjolras", "value": 1 },
//     { "source": "Montparnasse", "target": "Javert", "value": 1 },
//     { "source": "Montparnasse", "target": "Babet", "value": 2 },
//     { "source": "Montparnasse", "target": "Gueulemer", "value": 2 },
//     { "source": "Montparnasse", "target": "Claquesous", "value": 2 },
//     { "source": "Montparnasse", "target": "Valjean", "value": 1 },
//     { "source": "Montparnasse", "target": "Gavroche", "value": 1 },
//     { "source": "Montparnasse", "target": "Eponine", "value": 1 },
//     { "source": "Montparnasse", "target": "Thenardier", "value": 1 },
//     { "source": "Toussaint", "target": "Cosette", "value": 2 },
//     { "source": "Toussaint", "target": "Javert", "value": 1 },
//     { "source": "Toussaint", "target": "Valjean", "value": 1 },
//     { "source": "Child1", "target": "Gavroche", "value": 2 },
//     { "source": "Child2", "target": "Gavroche", "value": 2 },
//     { "source": "Child2", "target": "Child1", "value": 3 },
//     { "source": "Brujon", "target": "Babet", "value": 3 },
//     { "source": "Brujon", "target": "Gueulemer", "value": 3 },
//     { "source": "Brujon", "target": "Thenardier", "value": 3 },
//     { "source": "Brujon", "target": "Gavroche", "value": 1 },
//     { "source": "Brujon", "target": "Eponine", "value": 1 },
//     { "source": "Brujon", "target": "Claquesous", "value": 1 },
//     { "source": "Brujon", "target": "Montparnasse", "value": 1 },
//     { "source": "Mme.Hucheloup", "target": "Bossuet", "value": 1 },
//     { "source": "Mme.Hucheloup", "target": "Joly", "value": 1 },
//     { "source": "Mme.Hucheloup", "target": "Grantaire", "value": 1 },
//     { "source": "Mme.Hucheloup", "target": "Bahorel", "value": 1 },
//     { "source": "Mme.Hucheloup", "target": "Courfeyrac", "value": 1 },
//     { "source": "Mme.Hucheloup", "target": "Gavroche", "value": 1 },
//     { "source": "Mme.Hucheloup", "target": "Enjolras", "value": 1 }
//   ]
// }


// const graphData = {
//   "nodes": [
//     { "id": "centralPoint", "group": 1 },
//     { "id": "PointA", "group": 1 },
//     { "id": "PointB", "group": 1 },
//     { "id": "PointC", "group": 1 }
//   ],
//   "links": [
//     { "source": "centralPoint", "target": "PointA", "value": 1 },
//     { "source": "centralPoint", "target": "PointB", "value": 8 },
//     { "source": "PointB", "target": "PointC", "value": 10 }
//   ]
// }

// const graphData = {
//   "nodes": [
//     { "id": "centralPoint", "group": 1 },
//     { "id": "PointA", "group": 1 },
//     { "id": "PointB", "group": 1 },
//     { "id": "PointC", "group": 1 },
//     { "id": "PointD", "group": 1 }
//   ],
//   "links": [
//     { "source": "centralPoint", "target": "PointA", "value": 1 },
//     { "source": "centralPoint", "target": "PointB", "value": 8 },
//     { "source": "PointB", "target": "PointC", "value": 10 },
//     { "source": "PointC", "target": "PointD", "value": 10 }
//   ]
// }

const graphData = {
  "nodes": [
    { "id": "centralPoint", "group": 1 },
    { "id": "PointA", "group": 1 },
    { "id": "PointB", "group": 1 },
    { "id": "PointC", "group": 1 },
    { "id": "PointD", "group": 1 }
  ],
  "links": [
    { "source": "centralPoint", "target": "PointA", "value": 1 },
    { "source": "centralPoint", "target": "PointB", "value": 8 },
    { "source": "PointB", "target": "PointC", "value": 10 },
    { "source": "PointC", "target": "PointD", "value": 10 }
  ]
}

// Set up Three.js scene, camera, and renderer
const scene = new THREE.Scene();
scene.background = new THREE.Color(0xff8de1);
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.z = 20;
const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);

const threejsMap = ref<Node>()
// const threejsMap = ref<HTMLElement>();

const domElement = renderer.domElement;


// Add lighting
const directionalLight = new THREE.DirectionalLight(0xffffff, 1.4);
directionalLight.position.set(-2, 5, 4);
scene.add(directionalLight);

const ambientLight = new THREE.AmbientLight(0x404040, 10);
scene.add(ambientLight);

const cube = createCube();
scene.add(cube);
const plane = createPlane();
scene.add(plane);
const batchedMesh = createBatchedMesh();
scene.add(batchedMesh);


// Create controls
const controls = new OrbitControls(camera, renderer.domElement);
controls.enableDamping = true;


///////////////custom graph from json data
// Constants for the force simulation
//const springConstant = 0.005;
const springConstant = 0.010;
const restLength = 5;
const repulsionStrength = 0.1;
const dampingFactor = 0.85;
const maxVelocity = 0.5;
const minVelocityThreshold = 0.1;
const stabilizationFrames = 60;
let stabilizedFrames = 0;
let graphStabilized = false;

// Store nodes and edges
const nodeMap: { [key: string]: { mesh: THREE.Mesh, velocity: THREE.Vector3 } } = {}; // For quick lookups by ID
const nodes: { id: string, mesh: THREE.Mesh, velocity: THREE.Vector3 }[] = [];   // Store node data with positions and velocities

// Node geometry and material
const nodeGeometry = new THREE.SphereGeometry(0.5, 16, 16);
const nodeMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000 });



graphData.nodes.forEach((nodeData) => {
  const node = new THREE.Mesh(nodeGeometry, nodeMaterial);
  node.position.set(Math.random() * 10 - 5, Math.random() * 10 - 5, Math.random() * 10 - 5);
  scene.add(node);

  // Create a single object with shared references for both nodes and nodeMap
  const nodeEntry = { id: nodeData.id, mesh: node, velocity: new THREE.Vector3() };

  nodes.push(nodeEntry);              // Add to nodes array
  nodeMap[nodeData.id] = nodeEntry;    // Add to nodeMap by ID (shared reference)
});

// Edge material and buffer geometry
const edgeMaterial = new THREE.LineBasicMaterial({ color: 0x0000ff });
//const edgePositions = new Float32Array(graphData.links.length * 2 * 3); // 2 points per line
const edgePositions = new Float32Array(graphData.links.length * 2 * 3); // 2 points per line
const edgeGeometry = new THREE.BufferGeometry();
edgeGeometry.setAttribute('position', new THREE.BufferAttribute(edgePositions, 3));

// Create edges based on JSON data
const edges = new THREE.LineSegments(edgeGeometry, edgeMaterial);
scene.add(edges);

graphData.links.forEach((link, index) => {
  const sourceNode = nodeMap[link.source].mesh;
  const targetNode = nodeMap[link.target].mesh;
  const startIdx = index * 6;
  edgePositions.set([sourceNode.position.x, sourceNode.position.y, sourceNode.position.z,
  targetNode.position.x, targetNode.position.y, targetNode.position.z], startIdx);
});

// Update forces for the spring network
function updateSpringForces() {
  graphData.links.forEach(link => {
    const sourceNode = nodeMap[link.source];
    const targetNode = nodeMap[link.target];
    // Step 1: Calculate the direction vector between the connected nodes
    const direction = new THREE.Vector3().subVectors(targetNode.mesh.position, sourceNode.mesh.position);
    const distance = direction.length(); // Distance between the nodes
    // Step 2: Calculate displacement from rest length
    const displacement = distance - restLength;
    // Step 3: Calculate spring force magnitude based on displacement and spring constant
    const forceMagnitude = springConstant * displacement;
    // Step 4: Normalize the direction and scale it by the calculated force magnitude
    const force = direction.normalize().multiplyScalar(forceMagnitude);
    // Step 5: Apply the calculated force to each node's velocity in opposite directions
    sourceNode.velocity.add(force); // Pull sourceNode toward targetNode
    targetNode.velocity.sub(force); // Pull targetNode toward sourceNode
  });
}

// Update repulsive forces for all nodes
function updateRepulsiveForces() {
  for (let i = 0; i < nodes.length; i++) {
    for (let j = i + 1; j < nodes.length; j++) {
      const direction = new THREE.Vector3().subVectors(nodes[j].mesh.position, nodes[i].mesh.position);
      const distance = direction.length() + 0.01; // Add epsilon to avoid division by zero
      const forceMagnitude = repulsionStrength / (distance * distance);
      const force = direction.normalize().multiplyScalar(forceMagnitude);
      nodes[i].velocity.sub(force);
      nodes[j].velocity.add(force);
    }
  }
}

// Apply damping and cap velocity
function applyDampingAndCapVelocity() {
  nodes.forEach(node => {
    node.velocity.multiplyScalar(dampingFactor);
    if (node.velocity.length() > maxVelocity) {
      node.velocity.setLength(maxVelocity);
    }
  });
}

// Update node positions based on velocity
function updatePositions() {
  nodes.forEach(node => {
    // Add velocity to position
    node.mesh.position.add(node.velocity); // This should apply the accumulated velocity
  });
}

// Update edges to keep them connected to nodes
function updateEdges() {
  graphData.links.forEach((link, index) => {
    const sourceNode = nodeMap[link.source].mesh;
    const targetNode = nodeMap[link.target].mesh;

    const startIdx = index * 6;
    edgePositions.set([sourceNode.position.x, sourceNode.position.y, sourceNode.position.z,
    targetNode.position.x, targetNode.position.y, targetNode.position.z], startIdx);
  });
  edgeGeometry.attributes.position.needsUpdate = true;
}

// Check if the network has stabilized
function checkStabilization() {
  const allNodesStable = nodes.every(node => node.velocity.length() < minVelocityThreshold);
  if (allNodesStable) {
    stabilizedFrames += 1;
  } else {
    stabilizedFrames = 0;
  }
  if (stabilizedFrames >= stabilizationFrames) {
    graphStabilized = true; // Set the flag to stop further graph calculations
    console.log("Network stabilized.");
  }
}


onMounted(() => {
  threejsMap.value?.appendChild(domElement);
  window.addEventListener("resize", setSize);
  setSize();
  animate();
})

const setSize = () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(window.devicePixelRatio);
};


// Render the scene
function animate() {
  requestAnimationFrame(animate);

  // // Only update graph forces if it hasn't stabilized
  if (!graphStabilized) {
    updateRepulsiveForces();
    updateSpringForces();
    applyDampingAndCapVelocity();
    updatePositions();
    updateEdges();
    checkStabilization();
  }

  renderer.render(scene, camera);
  controls.update();
  TWEEN.update();
}

</script>
<style scoped></style>