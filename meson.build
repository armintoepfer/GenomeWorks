project(
  'UNANIMITY',
  ['cpp', 'c', 'cuda'],
  version : '0.0.99',
  default_options : [
    'buildtype=release',
    'warning_level=3',
    'cpp_std=c++14',
    'c_std=c11',
    'b_ndebug=if-release'],
  license : 'BSD-3',
  meson_version : '>= 0.52.0')

#########
# FLAGS #
#########

gw_cpp_arch_flags = []
cpp = meson.get_compiler('cpp')
foreach cflag: [
  '-Wduplicated-cond',
  '-Wduplicated-branches',
  '-Wlogical-op',
  '-Wrestrict',
  '-Wold-style-cast',
  '-Wuseless-cast',
  '-Wdouble-promotion',
  '-Wunused-local-typedef',
  '-Wunused-parameter',
  '-Wno-sign-compare',
  '-Wshadow',
  '-Wformat=1']
    if cpp.has_argument(cflag)
      gw_cpp_arch_flags += cflag
    endif
endforeach

gw_c_arch_flags = []
c = meson.get_compiler('c')
foreach cflag: [
  '-Wc++-compat',
  '-Wno-unused-parameter',
  '-Wno-unused-variable',
  '-Wno-sign-compare',
  '-Wno-unused-but-set-variable']
    if c.has_argument(cflag)
      gw_c_arch_flags += cflag
    endif
endforeach

cuda = import('unstable-cuda')
nvcc = meson.get_compiler('cuda')

add_project_arguments('-DGW_ENABLE_CACHING_ALLOCATOR', language : ['cpp', 'cuda'])

gw_cuda_arch_flags = cuda.nvcc_arch_flags(nvcc, 'Auto')
gw_cuda_arch_flags += '--expt-relaxed-constexpr'
if (get_option('b_ndebug') == 'true') or ((get_option('b_ndebug') == 'if-release') and (get_option('buildtype') == 'release'))
  gw_cuda_arch_flags += '-DNDEBUG'
endif
gw_cuda_arch_readable = cuda.nvcc_arch_readable(nvcc, 'Auto')
gw_driver_version = cuda.min_driver_version(nvcc.version())

message('NVCC version:   ' + nvcc.version())
message('NVCC flags:     ' + ' '.join(gw_cuda_arch_flags))
message('NVCC readable:  ' + ' '.join(gw_cuda_arch_readable))
message('Driver version: >=' + gw_driver_version)

#####################
# cuda dependencies #
#####################

cuda_versions = [
  '11.3',
  '11.2',
  '11.1',
  '11.0',
  '10.0',
  '9.2',
  '9.1',
  '9.0',
  '8.0',
  '7.5',
  '7.0',
  '6.5',
]
cuda_ver = ''

# FIXME: use break syntax when we use meson >= '0.49'
foreach v : cuda_versions
  if cuda_ver == ''
    cuda_dep = dependency('cuda-' + v, required: false)
    cudart_dep = dependency('cudart-' + v, required: false)
    if cuda_dep.found() and cudart_dep.found()
      cuda_ver = v
      break
    endif
  endif
endforeach

if cuda_dep.found()
  cuda_header_found = c.has_header('cuda.h', dependencies: cuda_dep)
  cuda_lib_found = c.has_function('cuInit', dependencies: cuda_dep)
  if not cuda_header_found or not cuda_lib_found
    message ('Missing required header and/or function in cuda dependency')
    cuda_dep = dependency('', required : false)
  endif
endif

if cudart_dep.found()
  cudart_header_found = c.has_header('cuda_runtime_api.h', dependencies: cudart_dep)
  cudart_lib_found = c.has_function('cudaGetErrorString', dependencies: cudart_dep)
  if not cudart_header_found or not cudart_lib_found
    message ('Missing required header and/or function in cudart dependency')
    cudart_dep = dependency('', required : false)
  endif
endif

if not cuda_dep.found() or not cudart_dep.found()
    message ('Missing cuda dependencies')
    cuda_dep = dependency('cuda-' + cuda_ver, required : true)
    cudart_dep = dependency('cudart-' + cuda_ver, required : true)
endif

gw_lib_deps = [
  cuda_dep,
  cudart_dep,
]

###########
# headers #
###########

if not meson.is_subproject()
  install_headers(
    files([
      'common/base/include/claraparabricks/genomeworks/gw_config.hpp',
      'common/base/include/claraparabricks/genomeworks/types.hpp',
    ]),
    subdir : 'claraparabricks/genomeworks')

  install_headers(
    files([
      'cudaaligner/include/claraparabricks/genomeworks/cudaaligner/aligner.hpp',
      'cudaaligner/include/claraparabricks/genomeworks/cudaaligner/alignment.hpp',
      'cudaaligner/include/claraparabricks/genomeworks/cudaaligner/cudaaligner.hpp',
    ]),
    subdir : 'claraparabricks/genomeworks/cudaaligner')

  install_headers(
    files([
      'common/base/include/claraparabricks/genomeworks/logging/logging.hpp',
    ]),
    subdir : 'claraparabricks/genomeworks/logging')

  install_headers(
    files([
      'common/base/include/claraparabricks/genomeworks/utils/allocator.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/cudasort.cuh',
      'common/base/include/claraparabricks/genomeworks/utils/cudautils.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/device_buffer.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/device_preallocated_allocator.cuh',
      'common/base/include/claraparabricks/genomeworks/utils/exceptions.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/genomeutils.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/graph.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/limits.cuh',
      'common/base/include/claraparabricks/genomeworks/utils/mathutils.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/pinned_host_vector.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/signed_integer_utils.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/stringutils.hpp',
      'common/base/include/claraparabricks/genomeworks/utils/threadsafe_containers.hpp',
    ]),
    subdir : 'claraparabricks/genomeworks/utils')
endif

cudaalign_include_directories = include_directories(['cudaaligner/include'])
common_include_directories = include_directories(['common/base/include'])

gw_include_directories = [
  cudaalign_include_directories,
  common_include_directories
]

###########
# sources #
###########

common_base_cpp_sources = files([
  'common/base/src/cudautils.cpp',
  'common/base/src/graph.cpp',
  'common/base/src/logging.cpp',
])

cudaalign_lib_cpp_sources = files([
  'cudaaligner/src/aligner.cpp',
  'cudaaligner/src/aligner_global.cpp',
  'cudaaligner/src/aligner_global_hirschberg_myers.cpp',
  'cudaaligner/src/aligner_global_myers_banded.cpp',
  'cudaaligner/src/aligner_global_myers.cpp',
  'cudaaligner/src/aligner_global_ukkonen.cpp',
  'cudaaligner/src/alignment.cpp',
  'cudaaligner/src/alignment_impl.cpp',
  'cudaaligner/src/cudaaligner.cpp',
  'cudaaligner/src/needleman_wunsch_cpu.cpp',
  'cudaaligner/src/ukkonen_cpu.cpp',
])

cudaalign_lib_cu_sources = files([
  'cudaaligner/src/hirschberg_myers_gpu.cu',
  'cudaaligner/src/aligner_global.cpp',
  'cudaaligner/src/myers_gpu.cu',
  'cudaaligner/src/ukkonen_gpu.cu',
])

gw_sources = [
  common_base_cpp_sources,
  cudaalign_lib_cpp_sources,
  cudaalign_lib_cu_sources,
]

###########
# library #
###########

# install library if
# - either running as a proper project
# - or using shared libraries
gw_lib_install = (not meson.is_subproject()) or (get_option('default_library') != 'static')

gw_lib = library(
  'genomeworks',
  gw_sources,
  install : gw_lib_install,
  # use boost SONAME practice:
  #   cause ld startup issues before
  #   you even have the chance of running
  #   into ABI issues.
  soversion : meson.project_version(),
  version : meson.project_version(),
  dependencies : gw_lib_deps,
  include_directories : gw_include_directories,
  cpp_args : gw_cpp_arch_flags,
  c_args : gw_c_arch_flags,
  cuda_args : gw_cuda_arch_flags,
  override_options : ['warning_level=2'],
)

###################
# dependency info #
###################

if (not meson.is_subproject())
  import('pkgconfig').generate(
    gw_lib,
    version : meson.project_version(),
    name : 'genomeworks',
    filebase : 'genomeworks',
    description : 'GPU-accelerated library for biological sequence analysis')
endif

genomeworks_dep = declare_dependency(
  include_directories : gw_include_directories,
  link_with : gw_lib,
  dependencies : gw_lib_deps,
  version : meson.project_version())
