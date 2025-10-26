

cmake_minimum_required(VERSION 3.22)

project(Synomix CXX)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)





find_package(spdlog REQUIRED)
find_package(absl REQUIRED)               
find_package(simdjson REQUIRED)
find_package(cctz REQUIRED)

find_package(Drogon REQUIRED)
find_package(Protobuf REQUIRED)
find_package(gRPC REQUIRED)

find_package(PQXX REQUIRED)               
find_package(SQLite3 REQUIRED)

find_package(Arrow REQUIRED)
find_package(Parquet REQUIRED)

find_package(Eigen3 REQUIRED)
find_package(TBB REQUIRED)                

find_package(GDAL REQUIRED)
find_package(GEOS REQUIRED)               
find_package(PROJ REQUIRED)


find_package(Qt6 REQUIRED COMPONENTS Core Gui Widgets Quick)


find_package(MapLibreNative QUIET CONFIG)
if(MapLibreNative_FOUND)

  set(MAPLIBRE_TARGET MapLibre::MapLibre)
else()

  find_package(mbgl QUIET CONFIG)
  if(mbgl_FOUND)
    set(MAPLIBRE_TARGET mbgl)
  endif()
endif()


find_package(uWebSockets QUIET)
find_package(cpr QUIET)
find_package(RdKafka QUIET)              


add_executable(synomix
  src/main.cpp
  src/api/MetricsCtrl.cpp
  src/core/Db.cpp
  src/compute/DerivedCalculator.cpp
  # add more .cpp here
)

target_compile_definitions(synomix PRIVATE
  SPDLOG_ACTIVE_LEVEL=SPDLOG_LEVEL_INFO
)

# Include your own headers
target_include_directories(synomix PRIVATE
  ${PROJECT_SOURCE_DIR}/include
)

# Link everything. Imported targets carry include dirs & transitive deps.
target_link_libraries(synomix PRIVATE
  # plumbing
  fmt::fmt
  spdlog::spdlog
  absl::strings absl::time
  simdjson::simdjson
  cctz::cctz

  # http/rpc
  Drogon::Drogon
  protobuf::libprotobuf
  gRPC::grpc++              # plus your generated *_grpc.pb.cc later

  # data stores
  pqxx                       # libpqxx
  SQLite::SQLite3

  # columnar
  Arrow::arrow
  Parquet::parquet

  # math/parallel
  Eigen3::Eigen
  TBB::tbb

  # geospatial
  GDAL::GDAL
  GEOS::geos_c
  PROJ::proj

  # ui (desktop 2D)
  Qt6::Core Qt6::Gui Qt6::Widgets Qt6::Quick

  # map renderer if found
  ${MAPLIBRE_TARGET}
)

# ---- Protobuf/gRPC codegen example (adjust to your .proto paths) ----
# set(PROTO_FILES proto/compute.proto)
# protobuf_generate(TARGET synomix PROTOS ${PROTO_FILES})
# grpc_generate_cpp(GRPC_SRCS GRPC_HDRS proto/compute.proto)
# target_sources(synomix PRIVATE ${GRPC_SRCS} ${GRPC_HDRS})


if(cpr_FOUND)
  target_link_libraries(synomix PRIVATE cpr::cpr)
endif()

if(uWebSockets_FOUND)
  target_link_libraries(synomix PRIVATE uWebSockets)
endif()

if(RdKafka_FOUND)
  target_link_libraries(synomix PRIVATE rdkafka++)
endif()

