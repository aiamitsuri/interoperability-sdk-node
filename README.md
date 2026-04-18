Basic Usage

    import * as wasm from "@aiamitsuri/interoperability-wrapper-wasm";
    
    interface FetchParams {
        page: string;
        [key: string]: any;
    }
    
    export class NodeSDKit {
    
        private async fetchInteroperability(params: FetchParams): Promise<string> {
            try {
                const res = await wasm.fetch_from_js(params);
                return JSON.stringify(res, null, 2);
            } catch (error) {
                throw new Error(`WASM Fetch failed: ${String(error)}`);
            }
        }
    
        public async runDemo(): Promise<void> {
            const params: FetchParams = { page: "1" };
    
            console.log("Node SDK");
    
            const response = await this.fetchInteroperability(params);
            console.log(response);
        }
    }
    
    const sdk = new NodeSDKit();
    await sdk.runDemo();

Dynamic Usage

    import * as wasm from "@aiamitsuri/interoperability-wrapper-wasm";
    
    interface FetchParams {
        page: string;
        [key: string]: any; 
    }
    
    interface Pagination {
        total_pages: number;
    }
    
    interface SDKItem {
        title: string;
    }
    
    interface FetchResponse {
        data: SDKItem[];
        pagination: Pagination;
    }
    
    class NodeSDKit {
        private async fetchInteroperability(params: FetchParams): Promise<FetchResponse> {
            const res = await wasm.fetch_from_js(params);
            return res as FetchResponse;
        }
    
        public async fetchPage(page: number): Promise<FetchResponse> {
            const params: FetchParams = { page: page.toString() };
            return await this.fetchInteroperability(params);
        }
    }
    
    async function main() {
        const sdk = new NodeSDKit();
        
        console.log("--- Bhilani Interop SDK ---");
    
        for (let pageNum = 1; pageNum <= 5; pageNum++) {
            try {
                const parsed = await sdk.fetchPage(pageNum);
                const totalPages = parsed.pagination.total_pages;
    
                if (parsed.data.length === 0 || pageNum > totalPages) {
                    console.log(`Page ${pageNum}: Success (No Data - Server has ${totalPages} pages)`);
                } else {
                    console.log(`Page ${pageNum}: Success`);
                    parsed.data.forEach(item => console.log(`  - Title: ${item.title}`));
                }
            } catch (e: any) {
                console.log(`Page ${pageNum}: Failed (Error: ${e.message})`);
            }
        }
    }
    
    main();
