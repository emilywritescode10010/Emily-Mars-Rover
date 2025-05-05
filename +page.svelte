<script>
    // STEP 1: Start The Script Block, this is where all our logic and state will go
    
        // STEP 2: Define Rover Options by creating an array that contains the available rover options
        const roverOptions = [
            {value: 'all', name: 'All Rovers'},
            {value: 'perseverance', name: 'Perseverance'},
            {value: 'curiosity', name: 'Curiosity'},
            {value: 'opportunity', name: 'Opportunity'},
            {value: 'spirit', name: 'Spirit'},
        ];
    
        // STEP 3: Define Camera Options by creating an array that lists all cameras used by the rovers; users can filter photos by these
        const cameraOptions = [
            {value: 'all', name: 'All Cameras'},
            {value: 'FHAZ', name: 'Front Hazard Avoidance Camera'},
            {value: 'RHAZ', name: 'Rear Hazard Avoidance Camera'},
            {value: 'MAST', name: 'Mast Camera'},
            {value: 'CHEMCAM', name: 'Chemistry and Camera Complex'},
            {value: 'MAHLI', name: 'Mars Hand Lens Imager'},
            {value: 'MARDI', name: 'Mars Descent Imager'},
            {value: 'NAVCAM', name: 'Navigation Camera'},
            {value: 'PANCAM', name: 'Panoramic Camera'},
            {value: 'MINITES', name: 'Miniature Thermal Emission Spectrometer'},
        ];
    
        // STEP 5: Create Form Variables which we will use to track what the user selects in the form
        let selectedRover = 'all';
        let selectedCamera = 'all';
        let startDate = '';
        let endDate = '';
    
        // STEP 6: Add a Helper Function that calculates a 7-day range to use if no date is selected
        function getLast7Days(){
            const today = new Date();
            const lastWeek = new Date(today);
            lastWeek.setDate(today.getDate() - 7);
            return {
                start: lastWeek.toISOString().split('T')[0],
                end: today.toISOString().split('T')[0]
            };
        }
    
        // STEP 7: Write the fetchPhotos Function that will talk to the NASA API; it builds a URL, fetches photos, and prints them to the console
        import { photoResults } from '$app/navigation';
        import { goto } from 'app/navigation';
        async function fetchPhotos(){
            let allPhotos = [];
            let rovers = selectedRover === 'all' ? ['perseverance', 'curiousity', 'opportunity', 'spirit'] : [selectedRover];
    
            if(!startDate && !endDate){
                for(const rover of rovers){
                    let url = `https://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/latest_photos?api_key=${import.meta.env.VITE_NASA_API_KEY}`;
                    if(selectedCamera !== 'all'){
                        url += `&camera=${selectedCamera}`;
                    }
                    const res = await fetch(url);
                    const data = await res.json();
                    allPhotos = allPhotos.concat(data.latest_photos || [])
                }
                return;
            }
    
            let {start, end} = {start: startDate, end: endDate};
            for(const rover of rovers){
                let url = `https://api.nasa.gov/mars-photos/api/v1/rovers/${rover}/photos?api_key=${import.meta.env.VITE_NASA_API_KEY}`;
                url += `&earth_date=${end}`;
                if(selectedCamera !== 'all'){
                    url += `&camera=${selectedCamera}`;
                }
                const res = await fetch(url);
                const data = await res.json();
                allPhotos = allPhotos.concat(data.photos || []);
            }
            // stores the data
            photoResults.set(allPhotos);
            // redirects to the results page
            goto('/results');
        }
    
    </script>
    
    <h1>Mars Rover Photo Search</h1>
    <!-- 
    STEP 8: Build the HTML Form 
    that users will interact with to select the rover, camera, and dates
    -->
    
    <form on:submit|preventDefault={fetchPhotos}>
        <label>
            Rover:
            <select bind:value={selectedRover}>
                {#each roverOptions as rover}
                    <option value ={rover.value}>{rover.name}</option>
                {/each}
            </select>
        </label>
        <br />
    
        <label>
            Camera:
            <select bind:value={selectedCamera}>
                {#each cameraOptions as camera}
                    <option value ={camera.value}>{camera.name}</option>
                {/each}
            </select>
        </label>
        <br />
    
        <label>
            Start Date:
            <input type="date" bind:value={startDate} />
        </label>
        <br />
    
        <label>
            End Date:
            <input type="date" bind:value={endDate} />
        </label>
        <br />
    
        <button type="submit">Search</button>
    </form>
    
    <p>
        If you leave everything blank, you'll get the most recent photos from all rovers and all cameras (last 7 days).
    </p>
    