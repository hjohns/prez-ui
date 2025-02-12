<script lang="ts" setup>
import { ref, onMounted } from "vue";
import { BlankNode, Literal, NamedNode } from "n3";
import type { AnnotatedPredicate, AnnotatedQuad, ListItem, RowPred } from "@/types";
import PropRow from "@/components/proptable/PropRow.vue";

const props = defineProps<{
    item: ListItem;
    properties: AnnotatedQuad[];
    blankNodes: AnnotatedQuad[];
    prefixes: {[token: string]: string};
    hiddenPreds: string[];
}>();

const rows = ref<RowPred[]>([]);

function getNamedNodeQname(n: NamedNode | AnnotatedPredicate): string {
    let qname = "";
    Object.entries(props.prefixes).forEach(([prefix, prefixIri]) => {
        if (n.value.startsWith(prefixIri)) {
            qname = prefix + ":" + n.value.split(prefixIri)[1];
        }
    });
    return qname;
}

function qname(s: string): string {
    if (s === "a") { // special handling for "a" as rdf:type
        return props.prefixes.rdf + "type";
    } else {
        const [prefix, pred] = s.split(":");
        return props.prefixes[prefix] + pred;
    }
}

function getAnnotation(predicate: AnnotatedPredicate, annotationPred: string): string | undefined {
    return predicate.annotations.find(annotation => annotation.predicate.value === qname(annotationPred))?.object.value;
}

function copyIri() {
    navigator.clipboard.writeText(props.item.iri.trim());
}

function buildRows(properties: AnnotatedQuad[]): RowPred[] {
    let propRows: {[uri: string]: RowPred} = {};
    properties.forEach(p => {
        propRows[p.predicate.value] ??= {
            iri: p.predicate.value,
            objs: [],
            qname: getNamedNodeQname(p.predicate),
            label: getAnnotation(p.predicate, "rdfs:label"),
            description: getAnnotation(p.predicate, "dcterms:description"),
            explanation: getAnnotation(p.predicate, "dcterms:provenance"),
            order: 0
        };

        propRows[p.predicate.value].objs.push({
            value: p.object.value,
            qname: p.object instanceof NamedNode ? getNamedNodeQname(p.object) : undefined,
            datatype: p.object instanceof Literal ? { value: p.object.datatype.value, qname: getNamedNodeQname(p.object.datatype) } : undefined,
            language: p.object instanceof Literal ? p.object.language : undefined,
            description: undefined,
            termType: p.object.termType,
            label: undefined,
            rows: p.object instanceof BlankNode ? buildRows(props.blankNodes.filter(p1 => p1.subject.id === p.object.id)) : []
        });
    });
    return Object.values(propRows).sort((a, b) => a.order - b.order);
}

onMounted(() => {
    const properties = props.properties.filter(p => !props.hiddenPreds.includes(p.predicate.value));
    rows.value = buildRows(properties);
});
</script>

<template>
    <h1 class="page-title">
        {{ props.item.title || props.item.iri }}
        <small class="iri">
            <span class="badge">IRI</span>
            <a :href="props.item.iri" target="_blank" rel="noopener noreferrer">{{ props.item.iri }}</a>
            <button class="btn outline sm" title="Copy IRI" @click="copyIri()"><i class="fa-regular fa-clipboard"></i></button>
        </small>
        <small class="type">
            <span class="badge">Type</span>
            <a :href="props.item.type" target="_blank" rel="noopener noreferrer">{{ props.item.type }}</a>
        </small>
    </h1>
    <slot name="map"></slot>
    <p v-if="!!props.item.description"><em>{{ props.item.description }}</em></p>
    <table>
        <slot name="top"></slot>
        <PropRow v-for="row in rows" v-bind="row" />
        <slot name="bottom"></slot>
    </table>
</template>

<style lang="scss" scoped>
@import "@/assets/sass/_variables.scss";

h1.page-title {
    margin-top: 0;
    margin-bottom: 8px;
    display: flex;
    flex-direction: column;
    gap: 4px;

    small {
        display: flex;
        flex-direction: row;
        align-items: center;
        gap: 4px;
        font-weight: normal;

        &.iri {
            font-size: 0.5em;
        }

        &.type {
            font-size: 0.45em;
        }

        .badge {
            font-size: 0.9em;
        }
    }
}

table {
    border-collapse: collapse;

    :deep(tr) {

        th, td {
            padding: 6px;
        }

        th {
            text-align: left;
            vertical-align: top;
        }
    }

    & > :deep(tr:nth-child(2n + 1)) {
        background-color: var(--tableBg);
    }
}
</style>