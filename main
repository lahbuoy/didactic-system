local Terrain = workspace:WaitForChild("Terrain")

local CFG = {
	Seed = 0,
	ChunkSize = 64,
	ChunkRes = 4,
	WorldSize = 8,
	MaxHeight = 120,
	SeaLevel = 0,

	Biomes = {
		Grassland = {
			BaseWeight = 0.60,
			BaseMult = 1.00,
			Color = Color3.fromRGB(80,160,80)
		},
		Mountain = {
			BaseWeight = 0.25,
			BaseMult = 1.80,
			Color = Color3.fromRGB(150,150,150)
		},
		Desert = {
			BaseWeight = 0.15,
			BaseMult = 0.70,
			Color = Color3.fromRGB(194,178,128)
		}
	},
}

-- UTIL

local function Shuffle(list, rng)
	for i = #list, 2, -1 do
		local j = rng:NextInteger(1, i)
		list[i], list[j] = list[j], list[i]
	end
end

local function ClearWorld()
	Terrain:Clear()
	local d = workspace:FindFirstChild("GeneratedDecor")
	if d then d:Destroy() end
end

local function NewDecorFolders()
	local root = Instance.new("Folder")
	root.Name = "GeneratedDecor"
	root.Parent = workspace

	local t = Instance.new("Folder")
	t.Name = "Trees"
	t.Parent = root

	local r = Instance.new("Folder")
	r.Name = "Rocks"
	r.Parent = root

	local s = Instance.new("Folder")
	s.Name = "Shrubs"
	s.Parent = root

	return root, t, r, s
end

-- SIMPLE PERLIN NOISE

local grad = {
	{1,1,0},{-1,1,0},{1,-1,0},{-1,-1,0},
	{1,0,1},{-1,0,1},{1,0,-1},{-1,0,-1},
	{0,1,1},{0,-1,1},{0,1,-1},{0,-1,-1}
}

local function Fade(t)
	return t*t*t*(t*(t*6 - 15) + 10)
end

local function Lerp(a,b,t)
	return a + (b-a)*t
end

local function Dot(g,x,y,z)
	return g[1]*x + g[2]*y + g[3]*z
end

local function BuildPerm(rng)
	local p = table.create(512)
	for i = 1,256 do p[i] = i end
	Shuffle(p, rng)
	for i = 257,512 do p[i] = p[i-256] end
	return p
end

local function MakeNoise(perm)
	return function(x,y,z)
		local X = math.floor(x) % 256
		local Y = math.floor(y) % 256
		local Z = math.floor(z) % 256

		x -= math.floor(x)
		y -= math.floor(y)
		z -= math.floor(z)

		local u = Fade(x)
		local v = Fade(y)
		local w = Fade(z)

		local A  = perm[X+1] + Y
		local AA = perm[A+1] + Z
		local AB = perm[A+2] + Z
		local B  = perm[X+2] + Y
		local BA = perm[B+1] + Z
		local BB = perm[B+2] + Z

		local function GR(ID, X, Y, Z)
			return Dot(grad[(perm[ID] % 12) + 1], X, Y, Z)
		end

		local x1 = GR(AA+1, x,   y,   z)
		local x2 = GR(BA+1, x-1, y,   z)
		local y1 = GR(AB+1, x,   y-1, z)
		local y2 = GR(BB+1, x-1, y-1, z)

		local x3 = GR(AA+2, x,   y,   z-1)
		local x4 = GR(BA+2, x-1, y,   z-1)
		local y3 = GR(AB+2, x,   y-1, z-1)
		local y4 = GR(BB+2, x-1, y-1, z-1)

		local lx1 = Lerp(x1, x2, u)
		local lx2 = Lerp(y1, y2, u)
		local ly1 = Lerp(x3, x4, u)
		local ly2 = Lerp(y3, y4, u)

		return Lerp(
			Lerp(lx1, lx2, v),
			Lerp(ly1, ly2, v),
			w
		)
	end
end

-- BIOME + HEIGHT
local function BuildBiomeSet(rng)
	local out = {}
	local total = 0

	for name, b in pairs(CFG.Biomes) do
		local jitterW = 0.7 + rng:NextNumber() * 0.55
		local jitterH = 0.85 + rng:NextNumber() * 0.30

		local weight = b.BaseWeight * jitterW
		total += weight

		out[#out+1] = {
			Name = name,
			Color = b.Color,
			Weight = weight,
			HeightMult = b.BaseMult * jitterH
		}
	end

	-- build cumulative lookup
	local cumulative = {}
	local sum = 0
	for _, b in ipairs(out) do
		sum += b.Weight
		cumulative[#cumulative+1] = {B = b, Max = sum}
	end

	return cumulative, total
end

local function PickBiome(biomeSet, maxW, noiseFn, x, y)
	local n = math.abs(noiseFn(x*0.01, y*0.01, 0))
	local t = n * maxW

	for _, b in ipairs(biomeSet) do
		if t <= b.Max then
			return b.B
		end
	end
	return biomeSet[1].B
end

local function ComputeHeight(noiseFn, x, y)
	local h =
		noiseFn(x*0.02, y*0.02, 0) * 0.52 +
		noiseFn(x*0.005, y*0.005, 0) * 0.33 +
		noiseFn(x*0.001, y*0.001, 0) * 0.15

	return h
end

-- DECORATION

local function PlaceTree(folder, rng, x, y, height, biome)
	local trunkH = rng:NextInteger(10,18)

	local trunk = Instance.new("Part")
	trunk.Size = Vector3.new(2, trunkH, 2)
	trunk.Anchored = true
	trunk.Material = Enum.Material.Wood
	trunk.Color = Color3.fromRGB(92,64,51)
	trunk.CFrame = CFrame.new(x, height + trunkH/2, y)

	local leaves = Instance.new("Part")
	leaves.Shape = Enum.PartType.Ball
	leaves.Size = Vector3.new(8,8,8)
	leaves.Anchored = true
	leaves.Material = Enum.Material.Grass
	leaves.Color = biome.Color
	leaves.CFrame = CFrame.new(x, height + trunkH + 2, y)

	local m = Instance.new("Model")
	m.Name = biome.Name .. "_Tree"
	trunk.Parent = m
	leaves.Parent = m
	m.PrimaryPart = trunk
	m.Parent = folder
end

local function PlaceRock(folder, rng, x, y, height)
	local s = rng:NextNumber(3,7)
	local p = Instance.new("Part")
	p.Size = Vector3.new(s, s * rng:NextNumber(0.6,1.4), s)
	p.Anchored = true
	p.Material = Enum.Material.Rock
	p.Color = Color3.fromRGB(120,120,120)
	p.CFrame = CFrame.new(x, height + p.Size.Y/2, y) * CFrame.Angles(0, math.rad(rng:NextInteger(0,360)), 0)
	p.Parent = folder
end

local function PlaceShrub(folder, rng, x, y, height, biome)
	local p = Instance.new("Part")
	p.Shape = Enum.PartType.Ball
	p.Size = Vector3.new(2,2,2)
	p.Anchored = true
	p.Material = Enum.Material.Grass
	p.Color = biome.Color
	p.CFrame = CFrame.new(
		x + rng:NextNumber(-1,1),
		height + 1,
		y + rng:NextNumber(-1,1)
	)
	p.Parent = folder
end

local function TryDecor(rng, biome, x, y, height, treeF, rockF, shrubF, sea)
	if height <= sea then return end
	local r = rng:NextNumber()

	if biome.Name == "Grassland" then
		if r < 0.06 then
			PlaceTree(treeF, rng, x, y, height, biome)
		elseif r < 0.18 then
			PlaceShrub(shrubF, rng, x, y, height, biome)
		elseif r < 0.24 then
			PlaceRock(rockF, rng, x, y, height)
		end

	elseif biome.Name == "Mountain" then
		if r < 0.12 then
			PlaceRock(rockF, rng, x, y, height)
		end

	elseif biome.Name == "Desert" then
		if r < 0.08 then
			PlaceRock(rockF, rng, x, y, height)
		end
	end
end

-- FILL

local block = Vector3.new(4,4,4)

local function FillColumn(x,y,h,biome,sea,treeF,rockF,shrubF,rng)
	for z = 1, h do
		local mat = Enum.Material.Grass

		if z < sea then
			mat = Enum.Material.Sand
		elseif biome.Name == "Mountain" then
			mat = Enum.Material.Rock
		elseif biome.Name == "Desert" then
			mat = Enum.Material.Sand
		end

		Terrain:FillBlock(CFrame.new(x,z,y), block, mat)
	end

	if h < sea then
		for z = h+1, sea do
			Terrain:FillBlock(CFrame.new(x,z,y), block, Enum.Material.Water)
		end
	end

	TryDecor(rng, biome, x, y, h, treeF, rockF, shrubF, sea)
end

-- MAIN GENERATION

local generating = false

local function Generate()
	if generating then return end
	generating = true

	ClearWorld()

	-- seed
	local seed = os.time() + math.random(1, 1e8)
	CFG.Seed = seed
	local rng = Random.new(seed)

	CFG.SeaLevel = rng:NextInteger(0, math.floor(CFG.MaxHeight * 0.15))

	local perm = BuildPerm(rng)
	local Noise = MakeNoise(perm)

	local biomeSet, maxW = BuildBiomeSet(rng)
	local root, treeF, rockF, shrubF = NewDecorFolders()

	-- chunk queue
	local queue = {}
	for cx = 0, CFG.WorldSize-1 do
		for cy = 0, CFG.WorldSize-1 do
			queue[#queue+1] = {cx, cy}
		end
	end
	Shuffle(queue, rng)

	task.spawn(function()
		for _, c in ipairs(queue) do
			local cx, cy = c[1], c[2]

			for x = 0, CFG.ChunkSize, CFG.ChunkRes do
				for y = 0, CFG.ChunkSize, CFG.ChunkRes do
					local wx = cx * CFG.ChunkSize + x
					local wy = cy * CFG.ChunkSize + y

					local biome = PickBiome(biomeSet, maxW, Noise, wx, wy)
					local h = ComputeHeight(Noise, wx, wy)
					h = math.clamp(math.floor(h * CFG.MaxHeight * biome.HeightMult), 1, CFG.MaxHeight)

					FillColumn(wx, wy, h, biome, CFG.SeaLevel, treeF, rockF, shrubF, rng)
				end
			end

			task.wait() -- safe for potato machines
		end

		generating = false
		print("Terrain generated:", seed)
	end)
end

-- TRIGGER

local prompt = workspace.interactscreen.screen.prompt
local debounce = false

prompt.Triggered:Connect(function(plr)
	if debounce or generating then return end
	debounce = true

	print("Regenerate:", plr.Name)
	Generate()

	task.delay(2, function()
		debounce = false
	end)
end)

Generate()
