<script>
	import ApiClient from '@/utils/ApiClient';
    import PageWrapper from "@/components/base/PageWrapper.svelte";
    let credentialsList= [];
    ApiClient.collection("credentials").getFullList({
                    batch: 500,
                    fields: "*:excerpt(200)",
                    requestKey: null,
                }).then(result => {
            credentialsList=result
        })
        .catch(error => {
            console.error("Async function rejected with:", error);
        });






     function verify(data){
        let produrl="https://api.myinvois.hasil.gov.my";
        let preprodurl="https://preprod-api.myinvois.hasil.gov.my"
        let urltoUse="";
        let fullurl="";
        if(data.environment=="PILOT"){
            urltoUse=preprodurl;
        }else{
            urltoUse=produrl;
        }

        fullurl=urltoUse+"/connect/token"

        

        const myHeaders = new Headers();
        myHeaders.append("Content-Type", "application/x-www-form-urlencoded");
        const urlencoded = new URLSearchParams();
        urlencoded.append("client_id", data.client_id);
        urlencoded.append("client_secret", data.client_secret_1);
        urlencoded.append("grant_type", "client_credentials");
        urlencoded.append("scope", "InvoicingAPI");

        const requestOptions = {
        method: "POST",
        headers: myHeaders,
        body: urlencoded,
        };

        fetch(fullurl, requestOptions)
        .then((response) => response.text())
        .then((result) =>{
            data.isValid=true;
            data.verified=true;
            ApiClient.collection("credentials").update(data.id,data).then(result => {
            console.log("success")
            })
            .catch(error => {
                console.error("Async function rejected with:", error);
            });

        })
        .catch((error) => console.error(error));

        
        
        }

        

</script>

<PageWrapper>
    <table>
        <thead>
            <tr>
                <th>Company</th>
                <th>advance Option</th>
                <th>Environment</th>
                <th>Tin</th>
                <th>Brn</th>
                <th>client id</th>
                <th>client secret 1</th>
                <th>client secret 2</th>
                <th>status</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody>
            {#each credentialsList as cred}
                <tr>
                    <td>{cred.company}</td>
                    <td><button  tabindex="0" type="button" class="thumb link-hint closable" on:click={verify(cred)}>Option</button></td>
                    <td>{cred.environment}</td>
                    <td>{cred.tin}</td>
                    <td>{cred.tin}</td>
                    <td>{cred.client_id}</td>
                    <td>{cred.client_secret_1}</td>
                    <td>{cred.client_secret_2}</td>
                    <td>
                        {#if cred.isValid}
                        <i class="ri-checkbox-circle-fill pass"></i>
                        {:else}
                        <i class="ri-close-circle-fill notpass"></i>
                        {/if}
                    </td>
                    <td>
                        {#if cred.isValid && cred.verified}
                        <p>verified</p>
                        {:else}
                        <button  tabindex="0" type="button" class="thumb link-hint closable" on:click={verify(cred)}>Verify</button>
                        {/if}
                    </td>
                </tr>
            {/each}
        </tbody>
    </table>
</PageWrapper>