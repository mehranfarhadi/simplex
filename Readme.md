# Simplex Method Implementation

This document provides a minimal explanation of a JavaScript implementation of the Simplex method, used for solving linear programming problems. The main function calculates the Simplex algorithm, and there are helper functions for displaying tables and performing matrix operations.

## Functions Overview

### getSimplex

The `getSimplex` function implements the Simplex algorithm, taking in the number of decision variables (`quantDec`), the number of constraints (`quantRes`), and a `choice` parameter for displaying results.

```javascript
function getSimplex(quantDec, quantRes, choice) {
    let hasil_array = getValue(quantDec, quantRes);
    hasil_array.push(getValues(quantDec, quantRes));

    let allTables = [];
    let tablesCount = 0;
    let stopConditionValue = 0;
    let iMax = $("#iMax").val() || 20;
    let bValues = [];
    let [variabel_awal, variabel_tipe] = staticTableVars(quantDec, quantRes);
    const jumlah_kolom = quantDec + quantRes + 1;
    const jumlah_baris = quantRes + 1;

    for (let i = 0; i < jumlah_baris; i++) {
        bValues.push(hasil_array[i][jumlah_kolom - 1]);
    }

    matrixToTable(hasil_array, "Initial", variabel_tipe, variabel_awal, jumlah_baris, allTables, tablesCount++);

    let hasNegativeOrPositive = false;
    do {
        const [lowerNumber, columnLowerNumber] = getLowerNumberAndColumn(hasil_array, jumlah_baris, jumlah_kolom);
        if (lowerNumber === 0) break;

        const [pivoRow, updatedVariabelAwal] = whoLeavesBase(hasil_array, columnLowerNumber, jumlah_kolom, jumlah_baris, variabel_awal);
        variabel_awal = updatedVariabelAwal;
        const pivoValue = hasil_array[pivoRow][columnLowerNumber];

        hasil_array = divPivoRow(hasil_array, jumlah_kolom, pivoRow, pivoValue);
        hasil_array = nullColumnElements(hasil_array, pivoRow, columnLowerNumber, jumlah_baris, jumlah_kolom);

        const funczValues = hasil_array[jumlah_baris - 1];
        hasNegativeOrPositive = funczValues.some((v) => v < 0);

        stopConditionValue++;
        if (stopConditionValue >= iMax || stopConditionValue === 3) break;

        if (hasNegativeOrPositive) {
            matrixToTable(hasil_array, `Iteration ${stopConditionValue}`, variabel_tipe, variabel_awal, jumlah_baris, allTables, tablesCount++);
        }
    } while (hasNegativeOrPositive);

    matrixToTable(hasil_array, "Final", variabel_tipe, variabel_awal, jumlah_baris, allTables, tablesCount);
    senseTable(hasil_array, variabel_tipe, variabel_awal, quantDec, bValues);

    $(".shows").append('<br><div class="row" align="center"><button type="button" id="showss" class="btn btn-primary" onclick="hides()">Show Tables</button></div>');

    if (choice === 1) {
        $(".container").append(allTables[stopConditionValue]);
    } else {
        allTables.forEach(table => $(".container").append(table));
    }

    printResults(hasil_array, quantDec, quantRes, jumlah_kolom, variabel_awal);
}
```

### matrixToTable

The `matrixToTable` function converts a matrix into an HTML table and appends it to the DOM.

```javascript
function matrixToTable(matriz, divName, head, base, jumlah_baris, allTables, aux) {
    $("#auxDiv").html(
        `<div class="row"><div id="divTable${divName}" class="offset-md-2 col-md-8 offset-md-2 table-responsive">
            <div class="row"><h3>جدول ${divName}</h3></div>
            <table id="table${divName}" class="table table-bordered"></table>
        </div></div>`
    );

    let table = $(`#table${divName}`);
    let matrizTable = matriz.map(row => row.slice());
    let headTable = head.slice();
    let baseTable = base.slice();

    $("#firstPhase").remove();
    $("#startInputs").hide();
    $("#stepByStep").remove();

    matrizTable.unshift(headTable);

    for (let i = 1, j = 0; i <= jumlah_baris; i++, j++) {
        matrizTable[i].unshift(baseTable[j]);
    }

    for (let i = 0; i < matrizTable.length; i++) {
        let row = $("<tr />");
        table.append(row);
        for (let j = 0; j < matrizTable[i].length; j++) {
            let cellValue = matrizTable[i][j];
            let cell;

            if (divName === "Final" && i === matrizTable.length - 1) {
                cell = cellValue < 0 ? $("<td>0</td>") : $("<td>" + Math.round(cellValue * 100) / 100 + "</td>");
            } else {
                cell = !isNaN(cellValue) ? $("<td>" + Math.round(cellValue * 100) / 100 + "</td>") : $("<td>" + cellValue + "</td>");
            }
            row.append(cell);
        }
    }

    allTables[aux] = $(`#divTable${divName}`)[0].outerHTML;
}
```

### printResults

The `printResults` function displays the optimal value and the basic variables.

```javascript
function printResults(matriz, quantDec, quantRes, jumlah_kolom, base) {
    let zValue = $("#min").is(":checked") ? matriz[matriz.length - 1][jumlah_kolom - 1] * -1 : matriz[matriz.length - 1][jumlah_kolom - 1];
    $("#results").append(`<div class="col-md-12">Optimal solution Z = ${Math.round(zValue * 100) / 100}</div><br>`);
    $("#results").append("<div> Basic Variables </div>");
    for (let i = 0; i < quantRes; i++) {
        let baseName = base[i];
        let baseValue = matriz[i][jumlah_kolom - 1];
        $("#results").append(`<div>${baseName} = ${Math.round(baseValue * 100) / 100}</div>`);
    }
}
```

### staticTableVars

The `staticTableVars` function initializes the basis and headers for the Simplex table.

```javascript
function staticTableVars(quantDec, quantRes) {
    let base = [];
    let head = [];

    for (let i = 0; i < quantRes; i++) {
        base.push(`R${i + 1}`);
    }
    base.push("Z");

    head.push("BV");
    for (let i = 0; i < quantDec; i++) {
        head.push(`X${i + 1}`);
    }
    for (let i = 0; i < quantRes; i++) {
        head.push(`R${i + 1}`);
    }
    head.push("RHs");

    return [base, head];
}
```

### nullColumnElements

The `nullColumnElements` function makes all elements in the pivot column zero except for the pivot row.

```javascript
function nullColumnElements(matriz, pivoRow, pivoColumn, jumlah_baris, jumlah_kolom) {
    for (let i = 0; i < jumlah_baris; i++) {
        if (i === pivoRow || matriz[i][pivoColumn] === 0) {
            continue;
        }
        let pivoAux = matriz[i][pivoColumn];
        for (let j = 0; j < jumlah_kolom; j++) {
            matriz[i][j] = matriz[pivoRow][j] * (pivoAux * -1) + matriz[i][j];
        }
    }
    return matriz;
}
```

### senseTable

The `senseTable` function performs sensitivity analysis on the final solution.

```javascript
function senseTable(matriz, head, base, quantDec, bValues) {
    let matrizTable = [];
    let headTable = [];
    let baseTable = [];

    let restNames = [];
    let restValues = [];
    let minMaxValues = [];

    for (let i = 0; i < matriz.length; i++) {
        matrizTable[i] = matriz[i].slice();
    }
    for (let i = 0; i < head.length; i++) {
        headTable[i] = head[i].slice();
    }
    for (let i = 0; i < base.length; i++) {
        baseTable[i] = base[i].slice();
    }

    matrizTable.unshift(headTable);

    for (let i = 1, j = 0; i <= matriz.length; i++, j++) {
        matrizTable[i].unshift(baseTable[j]);
    }

    for (let i = quantDec + 1, k = 0; i < matrizTable[0].length - 1; k++, i++) {
        restNames.push(matrizTable[0][i]);
        restValues.push(matrizTable[matrizTable.length - 1][i]);
        let auxArray = [];
        for (let j = 1; j < matrizTable.length - 1; j++) {
            let bCol = matrizTable[j][matrizTable[0].length - 1];
            let restCol = matrizTable[j][i];
            auxArray.push((bCol / restCol) * -1);
        }
        let minPos = Number.POSITIVE_INFINITY;
        let maxNeg = Number.NEGATIVE_INFINITY;
        for (let j = 0; j < auxArray.length; j++) {
            if (auxArray[j] > 0 && auxArray[j] < minPos) {
                minPos = auxArray[j];
            } else if (auxArray[j] < 0 && auxArray[j] > maxNeg) {
                maxNeg = auxArray[j];
            }
        }
        if (minPos === Number.POSITIVE_INFINITY) {
            minPos = 0;
        }
        if (maxNeg === Number.NEGATIVE_INFINITY) {
            maxNeg = 0;
        }
        minMaxValues.push([maxNeg + bValues[k], minPos + bValues[k]]);
    }

    let senseMatriz = [];
    for (let i = 0; i < matrizTable.length - 2; i++) {
        let auxArray = [];
        auxArray.push(restNames[i]);
        auxArray.push(restValues[i]);
        senseMatriz.push(auxArray);
    }
    for (let i = 0; i < senseMatriz.length; i++) {
        for (let j = 0; j < minMaxValues[0].length; j++) {
            senseMatriz[i].push(minMaxValues[i][j]);
        }
        senseMatriz[i].push(bValues[i]);
    }
    senseMatriz.unshift(["Resource", "Shadow Price", "Min", "Max", "Initial"]);

    $(".container").append(
        '<hr><div id="divSenseTable" class="offset-md-2 col-md-8 offset-md-2 table-responsive table-color"><div class="row"><h3>Sensitivity Table:</h3></div></div>'
    );
    $(".container").append(
        '<div class="row"><div id="divSenseTable" class="offset-md-2 col-md-8 offset-md-2 table-responsive table-color"><table id="senseTable" class="table table-bordered"></table></div></div><hr>'
    );
    let table = $("#senseTable");
    for (let i = 0; i < senseMatriz.length; i++) {
        let row = $("<tr />");
        table.append(row);
        for (let j = 0; j < senseMatriz[i].length; j++) {
            let cell = !isNaN(senseMatriz[i][j]) ? $("<td>" + Math.round(senseMatriz[i][j] * 100) / 100 + "</td>") : $("<td>" + senseMatriz[i][j] + "</td>");
            row.append(cell);
        }
    }
}
```

This concludes the overview of the functions used in the JavaScript implementation of the Simplex method.
}
]
}

