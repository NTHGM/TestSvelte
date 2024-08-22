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
    //     list: Array<PluralItem> = new Array();
    // }
    //plural2
    class Plural2 {
        name: String;
        untranslatable: String;
        quantity: String;
        text: String;
    }

    //string array 1
    // class StringArray {
    //     name: String;
    //     untranslatable: String;
    //     list: Array<String> = new Array<String>();
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
    let strL;

    let stra;
    let straT;
    let straL: Array<StringArray2> = new Array<StringArray2>();

    let plur;
    let plurT;
    let plurL: Array<Plural2> = new Array<Plural2>();

    function createTable(datas, id) {
        let container = document.getElementById(id);
        container.innerHTML = "";
        let hot = new Handsontable(container, {
            data: datas,
            rowHeaders: true,
            colHeaders: true,
            height: "auto",
            autoWrapRow: true,
            autoWrapCol: true,
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

    function load(hot, data) {
        hot.loadData(data);
    }

    function convertToArray(arr): Array<any> {
        let x = [];
        if (!arr) {
            return x;
        }

        if (arr.length == 1) {
            x = [...x, arr];
        } else {
            x = [...arr];
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
        strL = str?.map((item) => ({
            name: item["@_name"],
            untranslatable: item["@_untranslatable"] || "",
            text: item["#text"],
        }));

        //stra
        straL.slice(0, straL.length - 1);
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
        plurL.slice(0, straL.length - 1);
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
            str = jsonObj.resources["string"];
            stra = jsonObj.resources["string-array"];
            plur = jsonObj.resources["plurals"];
            maping();
            strT = createTable(strL, "example");
            plurT = createTable(plurL, "example1");
            straT = createTable(straL, "example2");
        };
        reader.readAsText(file);
    }

    function show() {
        //console.log(jsonObj.resources);

        // console.log(str);
        // console.log(strL);

        //console.log(stra);
        //console.log(straL);

        console.log(plur);
        console.log(plurL);

        //console.log(ori);
    }

    const builder = new XMLBuilder(options);
    let ori = null;
    function JSONtoXML() {
        ori = builder.build(jsonObj);
    }
</script>

<div class="sidenav">
    <div
        style="height:70px; text-align:center; 
	background-color:gray; padding:25px 0px;"
    >
        <input
            id="upload"
            type="file"
            style="display:none;"
            on:change={OnUploadFile}
        />
        <label for="upload" class="but">Upload</label>
    </div>
    {#each fileList as file, i}
        <button on:click={() => XMLtoJSON(file)}>{file.name}</button>
        <!-- <a
                style="padding:0px 2px; width: 30%;"
                href={fileURL[i].toString()}
                download="string.xml"
            >
                download
            </a> -->
    {/each}
</div>

<div class="content">
    <h2>Table</h2>
    <button on:click={show}>Show</button>
    <div class="ta">
        <div id="example"></div>
    </div>
    <div class="ta">
        <div id="example1"></div>
    </div>
    <div class="ta">
        <div id="example2"></div>
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
    }

    .sidenav a:hover {
        background-color: #ddd;
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

    .ta {
        padding: 2px;
    }
</style>
