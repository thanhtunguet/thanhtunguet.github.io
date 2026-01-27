---
title: "The Ghost Rows Mystery: A Tale of Excel Import Optimization"
date: 2026-01-27 11:40:00 +0700
categories: [Today I learned, Programming]
tags: ["Excel"]
pin: false
mermaid: false
---

## The Problem That Started It All

It was supposed to be a straightforward feature: allow users to import orders from Excel files into our order management system. Upload a file, parse it, validate the data, and save it to the database. Simple, right?

Wrong.

Our users started complaining that when they uploaded their Excel files, the entire browser would freeze for several seconds—sometimes more than 10 seconds. The UI became completely unresponsive. No loading spinners, no progress bars, just a frozen screen that made users think the application had crashed.

This wasn't just a minor inconvenience. In a business environment where users might import hundreds of orders at once, a 10+ second freeze was unacceptable.

## The First Fix: Moving to Web Workers

The obvious solution to UI blocking was to move the Excel parsing off the main thread. I implemented a Web Worker to handle the file processing:

```typescript
// Main thread stays responsive
const worker = new Worker(new URL('./excelParser.worker', import.meta.url));
worker.postMessage({ file });

// Worker does the heavy lifting
self.onmessage = async (e) => {
  const workbook = XLSX.read(arrayBuffer);
  const jsonData = XLSX.utils.sheet_to_json(worksheet);
  // ... process data
};
```

Problem solved! Or so I thought.

The UI no longer froze, and users could see a nice "Processing..." spinner. But there was still something wrong—the processing itself was taking way too long. For some files, users had to wait 15+ seconds staring at a spinner before seeing any results.

## The Investigation Begins

I decided to add detailed performance logging to understand where the time was going:

```typescript
console.log('[Excel Worker] Convert ArrayBuffer to binary string:', time1);
console.log('[Excel Worker] Parse Excel file:', time2);
console.log('[Excel Worker] Convert worksheet to JSON:', time3);
console.log('[Excel Worker] Process rows:', time4);
```

I tested with three different files from our users:

**File 1**: A 2.88MB file from a user who complained about slow processing  
**File 2**: A typical small file with 36 orders  
**File 3**: A test file with just 10 orders

The results were... confusing:

```
File 1: 2.88MB → Processing time: 13.4 seconds
File 2: 0.01MB → Processing time: 0.02 seconds
File 3: 0.01MB → Processing time: 0.01 seconds
```

The small files were lightning fast. But that 2.88MB file? Over 13 seconds. That seemed excessive for a file that wasn't even that large.

## The Breakthrough: Reading the Logs

I dove deeper into the performance breakdown:

```
[Excel Worker] Convert ArrayBuffer to binary string: 60.70ms
[Excel Worker] Parse Excel file: 456.00ms
[Excel Worker] Convert worksheet to JSON: 12,836.90ms ← 😱
[Excel Worker] Process rows: 19.50ms
```

Wait. **12.8 seconds** out of 13.4 seconds was spent in the `sheet_to_json()` call? That's 96% of the total time!

But here's what made me dig deeper—when I looked at the actual data being processed:

```
Total rows parsed: 1,048,562
Valid rows after filtering: 1,581
```

**1,048,562 rows?!** But the user only imported 1,581 orders!

## The "Aha!" Moment

I opened the Excel file manually and scrolled down. Row 1,582... empty. Row 1,583... empty. Row 10,000... empty. Row 100,000... still empty!

But Excel's row counter at the bottom showed: **1,048,576 rows**.

That number looked familiar. A quick Google search confirmed it: **1,048,576 is the maximum number of rows in an Excel spreadsheet**.

The file was marked as having over a million rows, but only the first ~1,500 contained actual data. The remaining 1,047,000+ rows were "ghost rows"—empty rows that existed in the file's metadata but contained no actual data.

And our code was dutifully trying to convert all 1,048,562 rows to JSON.

## Understanding the Root Cause

Excel files can accumulate ghost rows in several ways:

1. **Accidental formatting**: A user selects "all cells" and applies formatting, marking millions of empty rows as "used"
2. **Copy-paste operations**: Copying large ranges extends the file's "used range" even if most cells are empty
3. **Deleted data**: Data gets deleted but the file structure retains the row references
4. **Template files**: Pre-formatted templates with empty rows already marked as "used"

In our case, the user had likely copied data from another source, pasted it into a template, and Excel marked the entire range as used—all 1 million+ rows.

### Why This Destroys Performance

When you call `XLSX.utils.sheet_to_json()`, the library processes **every row in the file's range**, not just rows with data:

```typescript
// This converts ALL 1,048,562 rows to JSON objects
const jsonData = XLSX.utils.sheet_to_json(worksheet);

// Result: An array with 1,048,562 objects (mostly empty)
// Memory: ~500MB
// Time: 12.8 seconds
// Actual useful data: 1,581 rows
```

The performance breakdown showed:

| Step | Time | Percentage |
|------|------|------------|
| Read file | 9.70ms | 0.07% |
| Parse Excel structure | 456ms | 3.41% |
| **Convert to JSON** | **12,836ms** | **96.08%** ← The killer |
| Process valid rows | 19.50ms | 0.15% |

Converting a massive JSON array doesn't scale linearly—it grows exponentially with size because:
- Memory allocation for 1M+ objects
- Iteration over every row
- JSON stringification overhead
- Garbage collection pressure

## Designing the Solution

Now that I understood the problem, I needed a solution. But I had to balance several concerns:

1. **Performance**: Don't process millions of ghost rows
2. **Usability**: Most legitimate imports are < 10,000 orders
3. **User experience**: Show progress, allow cancellation
4. **Memory**: Don't load millions of rows into memory at once

### The Strategy: Three-Pronged Approach

After analyzing the performance data and understanding the problem, I designed a three-part solution:

#### 1. Set a Practical Row Limit

Our business context: users import order data. Even bulk imports rarely exceed a few thousand orders. I set a hard limit:

```typescript
const MAX_ROWS_TO_PROCESS = 10000;
```

**Reasoning**: 
- **Legitimate use cases**: 10k rows is generous for order imports
- **Ghost row protection**: Stops processing at 10k instead of 1M+
- **Clear expectations**: Users know the limit upfront
- **Performance guarantee**: Bounded processing time

If a file claims to have 1,048,562 rows, we process the first 10,000 and ignore the rest (which are likely ghost rows anyway).

#### 2. Read in Batches, Not All at Once

The original approach:

```typescript
// ❌ OLD: Convert entire sheet to JSON (1M+ rows at once)
const jsonData = XLSX.utils.sheet_to_json(worksheet);
// Result: 12.8 seconds, 500MB memory spike
```

The new approach:

```typescript
// ✅ NEW: Read in small batches
const BATCH_SIZE = 500;

// First, get just the headers
const headers = XLSX.utils.sheet_to_json(worksheet, { 
  range: 0,  // Just row 0
  header: 1 
})[0];

// Then read 500 rows at a time
for (let start = 1; start < totalRows; start += BATCH_SIZE) {
  const batch = XLSX.utils.sheet_to_json(worksheet, {
    range: start,  // Read from row 'start'
    header: headers
  });
  
  // Process this batch immediately
  const validRows = batch.filter(isValidRow);
  sendBatchToMainThread(validRows);
}
```

**Why this works**:
- **Smaller JSON conversions**: 500 rows instead of 1M+
- **Memory efficient**: Process and discard each batch
- **Progressive**: Send data as it's ready
- **Interruptible**: Can cancel between batches

#### 3. Progressive Status with Cancel Option

Users shouldn't stare at a generic "Processing..." spinner for 10+ seconds. They need to know:
- What's happening right now
- How much progress has been made
- The ability to cancel if it's taking too long

I implemented three distinct states:

**Stage 1: Reading File**
```
🔄 Đang đọc file...
[Cancel Button]
```

**Stage 2: Parsing Excel**
```
🔄 Đang xử lý 2,500 / 10,000 dòng hợp lệ (25%)
[Cancel Button]
```

**Stage 3: Complete**
```
✅ Import thành công 1,581 đơn hàng
```

The cancel button terminates the worker and clears all accumulated data:

```typescript
const handleCancel = () => {
  worker.terminate();  // Kill the worker
  allData = [];        // Clear received data
  setProgress(null);   // Hide progress UI
};
```

## Implementation: The Technical Details

Now for the fun part—actually building this thing. Here's how I implemented each piece:

### Worker Architecture

The worker needs to:
1. Detect ghost rows early by checking total rows vs the limit
2. Read headers first to get column structure
3. Process rows in batches
4. Send progress updates
5. Respect the 10k row limit

```typescript
// excelParser.worker.ts
const MAX_ROWS_TO_PROCESS = 10000;
const BATCH_SIZE = 500;

self.onmessage = async (e: MessageEvent<WorkerMessage>) => {
  const { file } = e.data;
  const startTime = performance.now();
  
  try {
    // Step 1: Read file
    const arrayBuffer = await file.arrayBuffer();
    const fileSizeMB = (file.size / (1024 * 1024)).toFixed(2);
    
    // Step 2: Parse Excel structure (not the data yet!)
    const workbook = XLSX.read(arrayBuffer, { type: 'array' });
    const worksheet = workbook.Sheets[workbook.SheetNames[0]];
    
    // Step 3: Check total rows from metadata
    const range = XLSX.utils.decode_range(worksheet['!ref'] || 'A1');
    const totalRows = range.e.r + 1;
    
    // Step 4: Apply row limit
    const rowsToProcess = Math.min(totalRows, MAX_ROWS_TO_PROCESS);
    
    if (totalRows > MAX_ROWS_TO_PROCESS) {
      console.warn(
        `⚠️ File có ${totalRows.toLocaleString()} dòng (có thể là ghost rows), ` +
        `chỉ xử lý ${MAX_ROWS_TO_PROCESS.toLocaleString()} dòng đầu`
      );
    }
    
    // Notify main thread we're starting to parse
    self.postMessage({
      type: 'parse_start',
      totalRows: rowsToProcess
    });
    
    // Step 5: Read headers only (row 0)
    const headerData = XLSX.utils.sheet_to_json(worksheet, {
      range: 0,
      header: 1
    });
    const headers = headerData[0] as string[];
    
    // Step 6: Process in batches
    let processedCount = 0;
    
    for (let startRow = 1; startRow < rowsToProcess; startRow += BATCH_SIZE) {
      const endRow = Math.min(startRow + BATCH_SIZE - 1, rowsToProcess - 1);
      
      // Read just this batch
      const batchData = XLSX.utils.sheet_to_json(worksheet, {
        range: startRow,
        header: headers,
        defval: null
      });
      
      // Validate and filter
      const validRows = batchData
        .map((row: any) => parseOrderRow(row))
        .filter((row): row is ParsedOrder => row !== null);
      
      processedCount += batchData.length;
      
      // Send batch to main thread
      self.postMessage({
        type: 'batch',
        data: validRows,
        progress: {
          current: processedCount,
          total: rowsToProcess - 1 // -1 for header
        }
      });
    }
    
    // Step 7: Complete
    const totalTime = performance.now() - startTime;
    self.postMessage({
      type: 'complete',
      totalTime,
      totalRows: processedCount
    });
    
  } catch (error) {
    self.postMessage({
      type: 'error',
      error: error instanceof Error ? error.message : 'Unknown error'
    });
  }
};
```

### Main Thread Handler

The main thread needs to:
1. Show appropriate status for each stage
2. Accumulate batches as they arrive
3. Update progress bar
4. Handle cancellation
5. Clean up when done

```typescript
// excelImportService.ts
export const processExcelFile = async (
  file: File,
  onProgress: (status: ImportStatus) => void,
  cancelSignal: { cancelled: boolean }
): Promise<ParsedOrder[]> => {
  
  return new Promise((resolve, reject) => {
    const worker = new Worker(
      new URL('../workers/excelParser.worker', import.meta.url)
    );
    
    let allData: ParsedOrder[] = [];
    let totalRows = 0;
    
    // Stage 1: Reading file
    onProgress({
      stage: 'reading',
      message: 'Đang đọc file...'
    });
    
    worker.onmessage = (e) => {
      const { type, data, progress, totalRows: total, error } = e.data;
      
      // Check if cancelled
      if (cancelSignal.cancelled) {
        worker.terminate();
        reject(new Error('Import cancelled by user'));
        return;
      }
      
      switch (type) {
        case 'parse_start':
          // Stage 2: Parsing started
          totalRows = total;
          onProgress({
            stage: 'parsing',
            message: `Đang xử lý file...`,
            total: totalRows
          });
          break;
          
        case 'batch':
          // Accumulate data
          allData = [...allData, ...data];
          
          // Update progress
          onProgress({
            stage: 'parsing',
            message: `Đang xử lý ${progress.current} / ${progress.total} dòng`,
            current: progress.current,
            total: progress.total,
            percentage: Math.round((progress.current / progress.total) * 100)
          });
          break;
          
        case 'complete':
          // Stage 3: Complete
          worker.terminate();
          onProgress({
            stage: 'complete',
            message: `Import thành công ${allData.length} đơn hàng`
          });
          resolve(allData);
          break;
          
        case 'error':
          worker.terminate();
          reject(new Error(error));
          break;
      }
    };
    
    worker.onerror = (error) => {
      worker.terminate();
      reject(error);
    };
    
    // Start processing
    worker.postMessage({ file });
  });
};
```

### UI Component

The React component ties it all together:

{% raw %}
```typescript
const ExcelImportDialog = () => {
  const [status, setStatus] = useState<ImportStatus | null>(null);
  const cancelSignal = useRef({ cancelled: false });
  
  const handleFileUpload = async (file: File) => {
    cancelSignal.current.cancelled = false;
    
    try {
      const orders = await processExcelFile(
        file,
        setStatus,
        cancelSignal.current
      );
      
      // Save to database...
      await saveOrders(orders);
      
    } catch (error) {
      if (error.message === 'Import cancelled by user') {
        toast.info('Import đã bị huỷ');
      } else {
        toast.error('Lỗi khi import: ' + error.message);
      }
    } finally {
      setStatus(null);
    }
  };
  
  const handleCancel = () => {
    cancelSignal.current.cancelled = true;
  };
  
  return (
    <div>
      {status && (
        <div className="import-progress">
          <Spinner />
          <div className="status-message">
            {status.message}
          </div>
          {status.percentage && (
            <div className="progress-bar">
              <div 
                className="progress-fill" 
                style={{ width: `${status.percentage}%` }}
              />
              <span>{status.percentage}%</span>
            </div>
          )}
          <button onClick={handleCancel} className="cancel-button">
            Huỷ
          </button>
        </div>
      )}
    </div>
  );
};
```
{% endraw %}

## The Results: Before and After

### Performance Comparison

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **File with ghost rows** (2.88MB, 1M+ rows) | 13.4s | 0.2s | **67x faster** |
| **Memory usage** | ~500MB | ~15MB | **33x less** |
| **UI blocking** | Yes (frozen) | No (responsive) | ✅ Fixed |
| **Progress visibility** | None | Real-time with % | ✅ Added |
| **Cancellable** | No | Yes | ✅ Added |

### Real-World Impact

**Before**:
```
User uploads file with ghost rows
→ UI freezes for 13+ seconds
→ No indication of progress
→ Can't cancel
→ User thinks app crashed
→ Bad experience
```

**After**:
```
User uploads file with ghost rows
→ UI stays responsive
→ Shows "Processing 2,500 / 10,000 rows (25%)"
→ Can cancel anytime
→ Processes in < 1 second
→ Clear success message
→ Great experience
```

### The Detailed Performance Breakdown

For a file with ghost rows (1,048,562 total rows, 1,581 valid):

**Before optimization**:

| Step | Time | % of Total |
|------|------|------------|
| Read file | 9.7ms | 0.07% |
| Parse Excel metadata | 456ms | 3.41% |
| **Convert to JSON** | **12,837ms** | **96.08%** |
| Validate rows | 19.5ms | 0.15% |
| **Total** | **13,380ms** | **100%** |

**After optimization**:

| Step | Time | % of Total |
|------|------|------------|
| Read file | 9.7ms | 4.5% |
| Parse Excel metadata | 156ms | 72.2% |
| Convert headers to JSON | 2ms | 0.9% |
| Convert batches to JSON (10k rows) | 42ms | 19.4% |
| Validate rows | 7ms | 3.2% |
| **Total** | **~216ms** | **100%** |

The bottleneck (JSON conversion) went from **12.8 seconds** to **42 milliseconds**.

## Key Learnings and Takeaways

### 1. File Metadata Can Lie

Never trust file metadata blindly. Excel files can claim to have 1M+ rows when they only contain hundreds of actual rows. Always validate and set practical limits based on your use case.

### 2. Batched Processing > Bulk Conversion

Converting large datasets to JSON in one operation is an anti-pattern:
- Memory spikes exponentially
- No progress indication
- Can't be cancelled
- Slower overall (JSON stringification doesn't scale linearly)

Batched processing gives you:
- Constant memory usage
- Real-time progress
- Cancellation points
- Often faster (smaller JSON operations are more efficient)

### 3. Performance Logging Is Essential

Without detailed timing logs, I never would have identified that 96% of the time was spent in `sheet_to_json()`. Always instrument your code when investigating performance issues.

### 4. Context Matters for Optimization

The 10k row limit works for our order import use case. Your limit might be different based on:
- What kind of data you're processing
- Your users' typical workflows
- Your memory constraints
- Your performance requirements

Don't cargo-cult solutions—understand your context and adapt accordingly.

### 5. User Experience Is More Than Just Speed

Even with the optimization, a large file might take a second or two. But the experience is dramatically better because:
- UI stays responsive (Web Worker)
- User sees progress (percentage bar)
- User has control (cancel button)
- User knows what's happening (clear status messages)

Sometimes perception is as important as actual performance.

## Recommendations for Similar Implementations

If you're building Excel import functionality, here's my advice:

### 1. Set Practical Row Limits

```typescript
// Don't do this
const data = XLSX.utils.sheet_to_json(worksheet); // Unlimited!

// Do this
const MAX_ROWS = 10000; // Based on your use case
const totalRows = Math.min(range.e.r + 1, MAX_ROWS);
```

### 2. Always Use Web Workers for File Processing

```typescript
// Any operation that might take > 50ms should be in a worker
const worker = new Worker(new URL('./fileProcessor.worker', import.meta.url));
```

### 3. Provide Progress and Control

```typescript
// Show what's happening
onProgress({ message: 'Processing 2,500 / 10,000 rows (25%)' });

// Let users cancel
cancelSignal.current.cancelled = true;
worker.terminate();
```

### 4. Read in Batches

```typescript
// Not all at once
const BATCH_SIZE = 500;
for (let start = 0; start < total; start += BATCH_SIZE) {
  const batch = readBatch(start, BATCH_SIZE);
  processBatch(batch);
}
```

### 5. Validate Early and Often

```typescript
// Check file size before processing
if (file.size > 50 * 1024 * 1024) { // 50MB
  throw new Error('File quá lớn');
}

// Check row count after parsing metadata
if (totalRows > MAX_ROWS) {
  console.warn(`Chỉ xử lý ${MAX_ROWS} dòng đầu`);
}
```

## Conclusion

The journey from "simple Excel import" to "optimized, user-friendly Excel import" taught me a valuable lesson: **performance issues often have surprising root causes**.

I started with a UI freeze problem, fixed it with Web Workers, then discovered a deeper issue with ghost rows consuming 96% of processing time. The solution wasn't to optimize the JSON conversion itself—it was to avoid converting millions of ghost rows in the first place.

By setting a practical row limit (10k), reading in batches (500 rows), and providing clear progress indicators, we transformed a 13-second operation into a sub-second one, reduced memory usage by 97%, and dramatically improved the user experience.

**The key insight**: Don't process more data than necessary. Excel files can lie about their row counts, and bulk operations don't scale. Read incrementally, validate continuously, and always give users visibility and control.

Now when users upload files with ghost rows, instead of a 13-second freeze, they see:
```
🔄 Đang xử lý 10,000 / 10,000 dòng (100%)
✅ Import thành công 1,581 đơn hàng
```

And that makes all the difference.

---

## Technical Specifications

- **Stack**: TypeScript, React, Web Workers, XLSX (SheetJS)
- **Row Limit**: 10,000 rows (configurable constant)
- **Batch Size**: 500 rows (configurable constant)
- **Target Processing Time**: < 500ms for typical files
- **Memory Target**: < 20MB for worker operations
- **UI States**: 3 (Reading, Parsing with progress, Complete)
- **Cancellable**: Yes, at any point

## Further Reading

- [SheetJS Documentation](https://docs.sheetjs.com/)
- [Web Workers API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)
- [Excel File Format Specifications](https://learn.microsoft.com/en-us/openspecs/office_standards/ms-xlsx/)
- [Performance Analysis Document](./excel-import-performance-analysis.md)
