load("//tensorflow/core/platform:rules_cc.bzl", "cc_library")
load("//third_party/mlir:tblgen.bzl", "gentbl")
load(
    "//tensorflow/core/platform/default:cuda_build_defs.bzl",
    "if_cuda_is_configured",
)
load(
    "@local_config_rocm//rocm:build_defs.bzl",
    "if_rocm_is_configured",
)
load("//tensorflow:tensorflow.bzl", "get_compatible_with_cloud")

package(
    default_visibility = ["//tensorflow/compiler/mlir/tools/kernel_gen:friends"],
    licenses = ["notice"],  # Apache 2.0
)

cc_library(
    name = "tf_framework_legalize_to_llvm",
    srcs = ["tf_framework_legalize_to_llvm.cc"],
    hdrs = ["rewriters.h"],
    deps = [
        "//tensorflow/compiler/mlir/tools/kernel_gen/ir:tf_framework_ops",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:LLVMDialect",
        "@llvm-project//mlir:LLVMTransforms",
        "@llvm-project//mlir:StandardOps",
        "@llvm-project//mlir:Support",
        "@llvm-project//mlir:Transforms",
    ],
)

cc_library(
    name = "bufferize",
    srcs = ["bufferize.cc"],
    hdrs = ["rewriters.h"],
    deps = [
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:Pass",
        "@llvm-project//mlir:SCFDialect",
        "@llvm-project//mlir:Support",
        "@llvm-project//mlir:Transforms",
    ],
)

cc_library(
    name = "embed_tf_framework",
    srcs = ["embed_tf_framework.cc"],
    hdrs = ["rewriters.h"],
    deps = [
        "//tensorflow/compiler/mlir/tools/kernel_gen/ir:tf_framework_ops",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:LLVMDialect",
        "@llvm-project//mlir:LLVMTransforms",
        "@llvm-project//mlir:StandardOps",
        "@llvm-project//mlir:Support",
        "@llvm-project//mlir:Transforms",
    ],
)

gentbl(
    name = "kernel_gen_passes_inc_gen",
    compatible_with = get_compatible_with_cloud(),
    tbl_outs = [("-gen-pass-decls -name KernelGen", "kernel_gen_passes.h.inc")],
    tblgen = "@llvm-project//mlir:mlir-tblgen",
    td_file = "passes.td",
    td_srcs = ["@llvm-project//mlir:PassBaseTdFiles"],
)

cc_library(
    name = "passes",
    srcs = [
        "buffer_reuse_pass.cc",
        "bufferize_pass.cc",
        "embed_memref_prints.cc",
        "embed_tf_framework_pass.cc",
        "gpu_kernel_to_blob_pass.cc",
        "materialize_broadcasts_pass.cc",
        "parallel_loops_to_sequential.cc",
        "same_shape_propagation.cc",
        "shape_to_descriptors_pass.cc",
        "tensorflow_abi_knowledge_propagation.cc",
        "tf_kernel_to_llvm_pass.cc",
        "unfuse_batch_norm_pass.cc",
    ],
    hdrs = ["passes.h"],
    copts = if_cuda_is_configured(["-DGOOGLE_CUDA=1"]) + if_rocm_is_configured(["-DTENSORFLOW_USE_ROCM=1"]),
    deps = [
        "//tensorflow/compiler/mlir/hlo:materialize_broadcasts",  # buildcleaner: keep
        "//tensorflow/compiler/mlir/hlo:unfuse_batch_norm",  # buildcleaner: keep
        "//tensorflow/compiler/xla/service:hlo_module_config",
        "//tensorflow/compiler/xla:debug_options_flags",
        "//tensorflow/compiler/xla:statusor",
        "//tensorflow/compiler/xla:status",
        "//tensorflow/compiler/xla/service/gpu:target_constants",
        "//tensorflow/compiler/xla/service/gpu/llvm_gpu_backend",
        "//tensorflow/core/platform:cuda_libdevice_path",
        "//tensorflow/core:lib",
        ":bufferize",
        ":embed_tf_framework",
        ":kernel_gen_passes_inc_gen",
        ":tf_framework_legalize_to_llvm",
        "@llvm-project//llvm:Support",
        "@llvm-project//mlir:GPUDialect",
        "@llvm-project//mlir:GPUToGPURuntimeTransforms",
        "@llvm-project//mlir:IR",
        "@llvm-project//mlir:LLVMDialect",
        "@llvm-project//mlir:LLVMTransforms",
        "@llvm-project//mlir:LinalgTransforms",
        "@llvm-project//mlir:LinalgOps",
        "@llvm-project//mlir:Pass",
        "@llvm-project//mlir:SCFDialect",
        "@llvm-project//mlir:Shape",
        "@llvm-project//mlir:Analysis",
        "@llvm-project//mlir:TargetNVVMIR",
        "@llvm-project//mlir:TargetROCDLIR",
        "@llvm-project//mlir:ShapeToStandard",
        "@llvm-project//mlir:SCFToStandard",
        "@llvm-project//mlir:ShapeTransforms",
        "@llvm-project//mlir:StandardOps",
        "@llvm-project//mlir:AllPassesAndDialects",
        "@llvm-project//mlir:Support",
        "@llvm-project//mlir:Transforms",
        "@llvm-project//llvm:TransformUtils",
        "//tensorflow/compiler/mlir/hlo",
        "//tensorflow/compiler/mlir/hlo:hlo_legalize_to_lhlo",
        "//tensorflow/compiler/mlir/hlo:lhlo",
        "//tensorflow/compiler/xla/service/gpu:stream_executor_util",
        "//tensorflow/compiler/mlir/tools/kernel_gen/ir:tf_framework_ops",
    ] + if_cuda_is_configured([
        "//tensorflow/stream_executor/gpu:asm_compiler",
    ]) + if_rocm_is_configured([
        "//tensorflow/core/platform:rocm_rocdl_path",
    ]),
)
