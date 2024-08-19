<script lang="ts">
    import { XMLParser } from "fast-xml-parser";
    import { XMLBuilder } from "fast-xml-parser";

    //parser-builder
    let jsonObj = null;
    let str = null;
    let stra = null;
    let plur = null;
    const options = {
        ignoreAttributes: false,
        attributeNamePrefix: "@_",
        allowBooleanAttributes: true,
        suppressBooleanAttributes: false,
        format: true,
    };

    function XMLtoJSON(event) {
        const file = event.target.files[0];
        if (file) {
            //parser
            let jsonObj = null;
            let str = null;
            let stra = null;
            let plur = null;
            const options = {
                ignoreAttributes: false,
                attributeNamePrefix: "@_",
                allowBooleanAttributes: true,
                suppressBooleanAttributes: false,
                format: true,
            };
            const reader = new FileReader();
            reader.onload = function (e) {
                const xmlContent = e.target.result;
                const parser = new XMLParser(options);
                jsonObj = parser.parse(xmlContent);
                str = jsonObj.resources["string"];
                stra = jsonObj.resources["string-array"];
                plur = jsonObj.resources["plurals"];
                JSONtoXML();
                str = str.map((item) => ({
                    name: item["@_name"],
                    untranslatable: item["@_untranslatable"] || "",
                    text: item["#text"],
                }));
                // console.log(jsonObj.resources);
                // console.log("String:");
                console.log(str);
                // console.log("STRA:");
                // console.log(stra);
                // console.log("Plural:");
                // console.log(plur);
            };
            reader.readAsText(file);
        }
    }

    const builder = new XMLBuilder(options);
    let ori = null;
    function JSONtoXML() {
        ori = builder.build(jsonObj);
        //console.log(ori);
    }
</script>

<input type="file" on:change={XMLtoJSON} />

<main>
    <!-- {#if str != null}
        <div class="ali">
            <h1>String</h1>
            <table>
                <thead>
                    <tr>
                        <th>Key</th>
                        <th>Value</th>
                        <th>Untranslatable</th>
                    </tr>
                </thead>
                <tbody>
                    {#each str as resource}
                        <tr>
                            <td>{resource["@_name"]}</td>
                            <td>{resource["#text"]}</td>
                            <td>
                                {#if resource["@_untranslatable"] != undefined}
                                    {resource["@_untranslatable"]}
                                {:else}{/if}
                            </td>
                        </tr>
                    {/each}
                </tbody>
            </table>
        </div>
    {/if}
    {#if stra != null}
        <div class="ali">
            <h1>String-Array</h1>
            <table>
                <thead>
                    <tr>
                        <th>Key</th>
                        <th>Item</th>
                        <th>Untranslatable</th>
                    </tr>
                </thead>
                <tbody>
                    {#each stra as resource}
                        <hr class="divide-solid" />
                        <tr style="padding: 16dp;">
                            <td>{resource["@_name"]}</td>
                            <td style="width: 30%;">
                                <ol>
                                    {#each resource["item"] as item, i}
                                        <li style="margin: 8dp;">
                                            {item}
                                        </li>
                                    {/each}
                                </ol>
                            </td>
                            <td style="text-align: center;">
                                {#if resource["@_untranslatable"] != undefined}
                                    {resource["@_untranslatable"]}
                                {:else}{/if}
                            </td>
                        </tr>
                        <hr class="divide-solid" />
                    {/each}
                </tbody>
            </table>
        </div>
    {/if}
    {#if plur != null}
        <div class="ali">
            <h1>Plurals</h1>
            <table>
                <thead>
                    <tr>
                        <th>Key</th>
                        <th>Item</th>
                        <th>Untranslatable</th>
                    </tr>
                </thead>
                <tbody>
                    {#each plur as resource}
                        <hr class="divide-solid" />
                        <tr style="padding: 16dp;">
                            <td>{resource["@_name"]}</td>
                            <td style="width: 30%;">
                                <ol>
                                    {#each resource["item"] as item, i}
                                        <li style="margin: 8dp;">
                                            <div>
                                                {item["@_quantity"]}: {item[
                                                    "#text"
                                                ]}
                                            </div>
                                        </li>
                                    {/each}
                                </ol>
                            </td>
                            <td style="text-align: center;">
                                {#if resource["@_untranslatable"] != undefined}
                                    {resource["@_untranslatable"]}
                                {:else}{/if}
                            </td>
                        </tr>
                        <hr class="divide-solid" />
                    {/each}
                </tbody>
            </table>
        </div>
    {/if} -->
</main>

<style>
    h1 {
        display: block;
        font-size: 2em;
        margin-top: 0.67em;
        margin-bottom: 0.67em;
        margin-left: 0;
        margin-right: 0;
        font-weight: bold;
    }
    table {
        align-self: center;
        text-align: center;
        padding: 32px;
    }

    ali {
        padding: 32px;
    }
</style>
