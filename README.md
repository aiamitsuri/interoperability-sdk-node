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
    
Concurrent Usage

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

class JVMSDKit {
    private async fetchInteroperability(params: FetchParams): Promise<string> {
        const res = await wasm.fetch_from_js(params);
        return JSON.stringify(res);
    }

    public async fetchPages(pageRange: number[]): Promise<PromiseSettledResult<string>[]> {
        const tasks = pageRange.map(async (page) => {
            await new Promise(resolve => setTimeout(resolve, Math.random() * (251 - 50) + 50));
            
            const controller = new AbortController();
            const timeoutId = setTimeout(() => controller.abort(), 5000);

            try {
                const params: FetchParams = { page: page.toString() };
                const result = await this.fetchInteroperability(params);
                clearTimeout(timeoutId);
                return result;
            } catch (error) {
                clearTimeout(timeoutId);
                throw error;
            }
        });

        return Promise.allSettled(tasks);
    }
}

async function main() {
    const sdk = new JVMSDKit();
    console.log("--- Bhilani Interop SDK (TS Concurrency) ---");

    const pageRange = [1, 2, 3, 4, 5];
    const results = await sdk.fetchPages(pageRange);

    results.forEach((result, index) => {
        const pageNum = index + 1;

        if (result.status === "fulfilled") {
            try {
                const parsed: FetchResponse = JSON.parse(result.value);
                const totalPages = parsed.pagination.total_pages;

                if (parsed.data.length === 0 || pageNum > totalPages) {
                    console.log(`Page ${pageNum}: Success (No Data - Server has ${totalPages} pages)`);
                } else {
                    console.log(`Page ${pageNum}: Success`);
                    parsed.data.forEach(item => console.log(`  - Title: ${item.title}`));
                }
            } catch (e: any) {
                console.log(`Page ${pageNum}: Success (JSON Parsing Failed: ${e.message})`);
            }
        } else {
            console.log(`Page ${pageNum}: Failed (${result.reason?.message || result.reason})`);
        }
    });
}

main();

<img width="946" height="442" alt="Screenshot (202)" src="https://github.com/user-attachments/assets/f48bbebe-df5f-48bc-8426-1888d02883c7" />
