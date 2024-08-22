<script lang="ts">
    import { XMLParser } from "fast-xml-parser";
    import { XMLBuilder } from "fast-xml-parser";
    import Handsontable from "handsontable";
    import "handsontable/dist/handsontable.full.min.css";
    //parser-builder
    //plural1
    // class PluralItem {
    //     quantity: String;
    //     text: String;
    // }
    // class Plural {
    //     name: String;
    //     untranslatable: String;
    //     item: Array<PluralItem> = new Array();
    // }
    //plural2
    class Plural2 {
        name: String;
        untranslatable: String;
        quantity: String;
        text: String;
    }

    enum TableType {
        String,
        StringArray,
        Plural,
    }

    //string array 1
    // class StringArray {
    //     name: String;
    //     untranslatable: String;
    //     item: Array<String> = new Array<String>();
    // }
    //string array 2
    class StringArray2 {
        name: String;
        untranslatable: String;
        text: String;
    }

    let fileURL: Array<String> = new Array();
    let fileList: Array<File> = new Array();

    let jsonObj;

    let str;
    let strT;
    let strL: Array<String> = new Array();
    let strSettings = {
        colHeaders: ["Key", "Untranslatable", "Value"],
        dataSchema: { name: null, untranslatable: null, text: null },
        rowHeaders: true,
        height: "auto",
        autoWrapRow: true,
        autoWrapCol: true,
        with: "auto",
        minSpareRows: 1,
    };

    let stra;
    let straT;
    let straL: Array<StringArray2> = new Array<StringArray2>();
    let straSettings = {
        colHeaders: ["Key", "Untranslatable", "Value"],
        dataSchema: { name: null, untranslatable: null, text: null },
        rowHeaders: true,
        height: "auto",
        autoWrapRow: true,
        autoWrapCol: true,
        with: "auto",
        minSpareRows: 1,
    };

    let plur;
    let plurT;
    let plurL: Array<Plural2> = new Array<Plural2>();
    let plurSettings = {
        colHeaders: ["Key", "Untranslatable", "Quantity", "Value"],
        dataSchema: {
            name: null,
            untranslatable: null,
            quantity: null,
            text: null,
        },
        rowHeaders: true,
        height: "auto",
        with: "auto",
        minSpareRows: 1,
        autoWrapRow: true,
        autoWrapCol: true,
    };

    function loadTable(datas, id, table, type) {
        if (table) {
            loadData(table, datas);
            return table;
        }
        let container = document.getElementById(id);
        let setting;
        switch (type) {
            case TableType.String:
                setting = strSettings;
                break;
            case TableType.Plural:
                setting = plurSettings;
                break;
            case TableType.StringArray:
                setting = straSettings;
                break;
        }

        let hot = new Handsontable(container, {
            data: datas,
            ...setting,
            licenseKey: "non-commercial-and-evaluation", // for non-commercial use only
        });
        //neu map lai thi can tham chieu den jsonObj.resources
        hot.addHook("afterChange", (changes) => {
            changes?.forEach(([row, prop, oldValue, newValue]) => {
                switch (datas) {
                    case strL:
                        const item = str[row];
                        item["@_" + prop] = newValue;
                        break;
                    case straL:
                        //add
                        break;
                    case plurL:
                        //add
                        break;
                }
            });
        });
        return hot;
    }

    function loadData(hot, data) {
        hot.loadData(data);
    }

    function convertToArray(arr): Array<any> {
        let x = [];
        if (!arr) {
            return x;
        }

        if (!arr.length) {
            x = [arr, ...x];
        } else {
            return arr;
        }
        return x;
    }

    function Url(file) {
        let x = URL.createObjectURL(file);
        fileURL = [...fileURL, x];
    }
    //de data phu hop voi bang
    function maping() {
        //str
        str = convertToArray(str);
        strL.splice(0, strL.length);
        strL = str?.map((item) => ({
            name: item["@_name"],
            untranslatable: item["@_untranslatable"] || "",
            text: item["#text"],
        }));

        //stra
        stra = convertToArray(stra);
        straL.splice(0, straL.length);
        if (stra)
            stra.forEach((e) => {
                e["item"].forEach((element) => {
                    let tmp = new StringArray2();
                    tmp.name = e["@_name"];
                    tmp.untranslatable = e["@_untranslatable"] || "";
                    tmp.text = element;
                    straL = [...straL, tmp];
                });
            });

        //plural
        plur = convertToArray(plur);
        plurL.splice(0, straL.length);
        if (plur)
            plur.forEach((e) => {
                e["item"].forEach((element) => {
                    let tmp = new Plural2();
                    tmp.name = e["@_name"];
                    tmp.untranslatable = e["@_untranslatable"] || "";
                    tmp.quantity = element["@_quantity"];
                    tmp.text = element["#text"];
                    plurL = [...plurL, tmp];
                });
            });
    }

    function OnUploadFile(event) {
        const file = event.target.files[0];
        if (file) {
            fileList = [...fileList, file];
            XMLtoJSON(file);
            Url(file);
        }
        event.target.value = null;
    }

    //parser-builder
    const options = {
        ignoreAttributes: false,
        attributeNamePrefix: "@_",
        allowBooleanAttributes: true,
        suppressBooleanAttributes: false,
        format: true,
    };
    const parser = new XMLParser(options);

    function XMLtoJSON(file) {
        const reader = new FileReader();
        reader.onload = function (e) {
            const xmlContent = e.target.result;
            jsonObj = parser.parse(xmlContent);
            let str = jsonObj.resources["string"];
            let stra = jsonObj.resources["string-array"];
            let plur = jsonObj.resources["plurals"];
            maping();
            strT = loadTable(strL, "stringT", strT, TableType.String);
            plurT = loadTable(plurL, "pluralT", plurT, TableType.Plural);
            straT = loadTable(
                straL,
                "stringarrayT",
                straT,
                TableType.StringArray,
            );
            console.log(jsonObj);
        };
        reader.readAsText(file);
    }

    const builder = new XMLBuilder(options);
    let ori = null;
    function JSONtoXML() {
        ori = builder.build(jsonObj);
    }

    function openTab(evt, type) {
        let i, tabcontent;
        tabcontent = document.getElementsByClassName("tabcontent");
        for (i = 0; i < tabcontent.length; i++) {
            tabcontent[i].style.display = "none";
        }
        document.getElementById(type).style.display = "block";
    }
</script>

<div class="sidenav">
    <div
        style="height:70px; text-align:center; 
	background-color:gray; padding:25px 0px;"
    >
        <input
            on:change={OnUploadFile}
            id="upload"
            type="file"
            style="display:none;"
        />
        <label for="upload" class="but">Upload</label>
    </div>
    <div>
        <button on:click={(event) => openTab(event, "string")} id="defaultOpen"
            >String</button
        >
        <button on:click={(event) => openTab(event, "plural")}>Plural</button>
        <button on:click={(event) => openTab(event, "stringarray")}
            >String-Array</button
        >
    </div>
</div>

<div class="content">
    <div id="string" class="tabcontent">
        <h2>String</h2>
        <div id="stringT"></div>
    </div>

    <div id="plural" class="tabcontent">
        <h2>Plural</h2>
        <div id="pluralT"></div>
    </div>

    <div id="stringarray" class="tabcontent">
        <h2>String-Array</h2>
        <div id="stringarrayT"></div>
    </div>
</div>

<style>
    * {
        box-sizing: border-box;
    }

    .sidenav {
        height: 100%;
        width: 200px;
        position: fixed;
        z-index: 1;
        top: 0;
        left: 0;
        background-color: #111;
        overflow-x: hidden;
    }

    .sidenav button {
        color: white;
        padding: 16px;
        text-decoration: none;
        display: block;
        width: 100%;
    }

    .sidenav button:hover {
        background-color: #ddd;
        color: black;
    }
    .sidenav button:active {
        background-color: #924040;
        color: black;
    }
    .content {
        margin-left: 200px;
        padding-left: 20px;
    }

    .but {
        border: 2px solid black;
        border-radius: 5px;
        text-align: center;
        width: 40%;
        background-color: white;
        padding: 2px;
    }

    .but:hover {
        background-color: black;
        color: white;
        border: 2px solid white;
    }
    .but:active {
        background-color: gray;
        color: white;
        border: 2px solid white;
    }

    .tabcontent {
        display: none;
    }
</style>
