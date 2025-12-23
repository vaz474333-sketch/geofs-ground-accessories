(function waitForGeoFS() {

    if (!window.geofs || !geofs.api || !geofs.api.rendering || !geofs.aircraft?.instance) {
        console.log("[GeoFS Accessories] Waiting for GeoFS...");
        return setTimeout(waitForGeoFS, 1000);
    }

    (function () {

        const scene = geofs.api.rendering.scene;

    const Cesium = geofs.api.rendering.Cesium;

    console.log("[GeoFS Accessories] Loaded");

    let stairsFront = null;
    let stairsRear = null;
    let gpuUnit = null;
    let fuelTruck = null;

    function isWideBody() {
        const ac = geofs.aircraft.instance;
        return ac?.dimensions?.length > 55;
    }

    function getOffsets() {
        const ac = geofs.aircraft.instance;
        const heading = ac.heading * Math.PI / 180;

        return {
            front: {
                lat: Math.cos(heading) * 0.00006,
                lon: Math.sin(heading) * 0.00006
            },
            rear: {
                lat: -Math.cos(heading) * 0.00006,
                lon: -Math.sin(heading) * 0.00006
            }
        };
    }

    function spawnStairs() {
        removeStairs();

        const pos = geofs.aircraft.instance.llaLocation;
        const off = getOffsets();

        stairsFront = scene.entities.add({
            position: Cesium.Cartesian3.fromDegrees(
                pos[1] + off.front.lon,
                pos[0] + off.front.lat,
                pos[2]
            ),
            model: {
                uri: "https://raw.githubusercontent.com/vaz474333-sketch/geofs-ground-accessories/main/models/stairs.glb",
                scale: 1
            }
        });

        if (isWideBody()) {
            stairsRear = scene.entities.add({
                position: Cesium.Cartesian3.fromDegrees(
                    pos[1] + off.rear.lon,
                    pos[0] + off.rear.lat,
                    pos[2]
                ),
                model: {
                    uri: "https://raw.githubusercontent.com/vaz474333-sketch/geofs-ground-accessories/main/models/stairs.glb",
                    scale: 1
                }
            });
        }
    }

    function spawnGPU() {
        removeGPU();

        const pos = geofs.aircraft.instance.llaLocation;

        gpuUnit = scene.entities.add({
            position: Cesium.Cartesian3.fromDegrees(
                pos[1] + 0.00003,
                pos[0] - 0.00003,
                pos[2]
            ),
            model: {
                uri: "https://raw.githubusercontent.com/vaz474333-sketch/geofs-ground-accessories/main/models/gpu.glb",
                scale: 1
            }
        });
    }

    function spawnFuel() {
        removeFuel();

        const pos = geofs.aircraft.instance.llaLocation;
        const dist = isWideBody() ? 0.00015 : 0.0001;

        fuelTruck = scene.entities.add({
            position: Cesium.Cartesian3.fromDegrees(
                pos[1] - dist,
                pos[0],
                pos[2]
            ),
            model: {
                uri: "https://raw.githubusercontent.com/vaz474333-sketch/geofs-ground-accessories/main/models/fuel_truck.glb",
                scale: 1
            }
        });
    }

    function removeStairs() {
        if (stairsFront) scene.entities.remove(stairsFront);
        if (stairsRear) scene.entities.remove(stairsRear);
        stairsFront = stairsRear = null;
    }

    function removeGPU() {
        if (gpuUnit) scene.entities.remove(gpuUnit);
        gpuUnit = null;
    }

    function removeFuel() {
        if (fuelTruck) scene.entities.remove(fuelTruck);
        fuelTruck = null;
    }

    function removeAll() {
        removeStairs();
        removeGPU();
        removeFuel();
    }

    const btn = document.createElement("div");
    btn.innerText = "Accessories";
    btn.style.cssText = `
        position:absolute;
        left:10px;
        top:300px;
        padding:8px 14px;
        background:rgba(0,0,0,0.65);
        color:white;
        font-family:Arial;
        border-radius:4px;
        cursor:pointer;
        z-index:9999;
    `;
    document.body.appendChild(btn);

    const menu = document.createElement("div");
    menu.style.cssText = `
        position:absolute;
        left:120px;
        top:300px;
        background:rgba(20,20,20,0.85);
        padding:10px;
        border-radius:6px;
        display:none;
        color:white;
        font-family:Arial;
        z-index:9999;
        min-width:160px;
    `;
    menu.innerHTML = `
        <b>Ground Accessories</b><br><br>
        <div class="acc">🛫 Stairs</div>
        <div class="acc">🔌 GPU</div>
        <div class="acc">⛽ Fuel Truck</div>
        <hr>
        <div class="acc">❌ Remove Stairs</div>
        <div class="acc">❌ Remove GPU</div>
        <div class="acc">❌ Remove Fuel</div>
        <hr>
        <div class="acc">❌ Remove All</div>
    `;
    document.body.appendChild(menu);

    const actions = [
        spawnStairs,
        spawnGPU,
        spawnFuel,
        removeStairs,
        removeGPU,
        removeFuel,
        removeAll
    ];

    menu.querySelectorAll(".acc").forEach((el, i) => {
        el.onclick = actions[i];
        el.style.cursor = "pointer";
        el.style.padding = "4px";
    });

    btn.onclick = () => {
        menu.style.display = menu.style.display === "none" ? "block" : "none";
    };

    })();
})();


