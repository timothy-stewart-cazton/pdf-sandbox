<template>
    <div ref="pdfContainer" class="pdfContainer">
    </div>
</template>

<style>
    .pdfContainer {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
    }
    .pdfContainer .pageContainer {
        position: relative;
    }

    .pdfContainer .pageContainer canvas {
        border: 1px solid black;
    }

    .pdfContainer .pageContainer .highlight {
        position: absolute;
        background: yellow;
        opacity: 0.4;
    }
</style>

<script setup lang="ts">
    import { onMounted, ref } from 'vue';

    import * as PDFJS from 'pdfjs-dist';
    import 'pdfjs-dist/web/pdf_viewer.css';

    export interface Highlight {
        page: number;
        pageBbox: string;
        highlightBbox: string;
    }

    export interface Props {
        src?: string;
        pageBbox?: string;
        higlightBbox?: string[];
        highlights?: Highlight[];
    }

    // the version here needs to match pdfjs-dist
    PDFJS.GlobalWorkerOptions.workerSrc = '//cdnjs.cloudflare.com/ajax/libs/pdf.js/3.4.120/pdf.worker.min.js';

    const props = defineProps<Props>();

    const pdfContainer = ref<HTMLDivElement | null>(null);

    onMounted(async () => {
        const src = props.src;

        if (!src) {
            return;
        }
        
        if (!pdfContainer.value) return;

        try {
            const pdf = await PDFJS.getDocument(src).promise;

            await renderDocument(
                pdf, pdfContainer.value, props.highlights || [])
        } catch (error) {
            console.error('Error rendering PDF: ', error);
        }
    });

    async function renderDocument(
        pdf: PDFJS.PDFDocumentProxy, 
        container: HTMLDivElement,
        highlights: Highlight[]) 
    {
        const numPages = pdf.numPages;

        let promises = [];

        for (let pageNum = 1; pageNum <= numPages; pageNum++) {
            const pageHighlights = highlights
                .filter((x) => x.page === pageNum);

            const { promise } = renderPage(
                pdf, pageNum, pdfContainer.value!, pageHighlights);

            promises.push(promise);
        }

        await Promise.all(promises);
    }

    function renderPage(
        pdf: PDFJS.PDFDocumentProxy, 
        pageNum: number,
        pdfContainer: HTMLDivElement,
        highlights: Highlight[]): {
            pageContainer: HTMLDivElement,
            promise: Promise<void>
        }
    {
        const pageContainer = document.createElement('div');
        pageContainer.classList.add('pageContainer');
        pdfContainer.appendChild(pageContainer);

        const canvas = document.createElement('canvas');
        pageContainer.appendChild(canvas);

        async function renderAsync() {
            const canvasContext = canvas.getContext('2d');

            if (!canvasContext) {
                throw 'Canvas context is null or undefined';
            }

            const pageProxy = await pdf.getPage(pageNum);
            
            const viewport = pageProxy.getViewport({ 
                scale: 1.92
            });

            canvas.height = viewport.height;
            canvas.width = viewport.width;

            const renderPromise = pageProxy.render({ 
                canvasContext, 
                viewport 
            }).promise;

            for (const item of highlights) {
                let pageBbox = parseBboxString(item.pageBbox);
                let highlightBbox = parseBboxString(item.highlightBbox);

                const scaleX = (viewport.width / (pageBbox.endX - pageBbox.startX));
                const scaleY = (viewport.height / (pageBbox.endY - pageBbox.startY));

                const startX = highlightBbox.startX * scaleX;
                const endX = highlightBbox.endX * scaleX;
                const startY = highlightBbox.startY * scaleY;
                const endY = highlightBbox.endY * scaleY;

                const el = document.createElement('div');
                el.classList.add('highlight');
                pageContainer.appendChild(el);

                el.style.top = `${Math.ceil(startY)}px`;
                el.style.left = `${Math.ceil(startX)}px`;
                el.style.width = `${Math.ceil(endX - startX)}px`;
                el.style.height = `${Math.ceil(endY - startY)}px`;
            }

            await renderPromise;
        }

        return {
            pageContainer,
            promise: renderAsync()
        };
    };

    function parseBboxString(str: string): ParsedBBOX {
        const split = str.split(' ').filter((x) => x.length > 0);

        return {
            startX: parseFloat(split[0]),
            startY: parseFloat(split[1]),
            endX: parseFloat(split[2]),
            endY: parseFloat(split[3]),
        };
    }

    interface ParsedBBOX {
        startX: number;
        startY: number;
        endX: number;
        endY: number;
    }
</script>