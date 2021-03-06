---
title: struct x264_t分析
date: 2017-04-28 01:54:20
tags:
---
x264_t是一个控制X264编码的全局性结构体，该结构体控制着视频一帧一帧的编码，包括中间参考帧管理、码率控制、全局参数等一些重要参数和结构体。接下来我们详细分析该结构体的元素 <!-- more -->该结构体定义在common/common.h中：

```c
struct x264_t
{
	/* encoder parameters */
	x264_param_t    param;		// 编码器编码参数，包括量化步长、编码级别等等一些参数

	x264_t   *thread[X264_THREAD_MAX+1];
	x264_t   *lookahead_thread[X264_LOOKAHEAD_THREAD_MAX];
	int    b_thread_active;
	int    i_thread_phase; 			/* which thread to use for the next frame */
	int    i_thread_idx;   			/* which thread this is */
	int    i_threadslice_start; 		/* first row in this thread slice */
	int    i_threadslice_end; 		/* row after the end of this thread slice */
	int    i_threadslice_pass; 		/* which pass of encoding we are on */
	x264_threadpool_t  *threadpool;
	x264_threadpool_t  *lookaheadpool;
	x264_pthread_mutex_t mutex;
	x264_pthread_cond_t   cv;

	 /* bitstream output */
	struct
	{
		int    i_nal;
		int    i_nals_allocated;
		int    i_bitstream;    		/* size of p_bitstream */
		uint8_t   *p_bitstream;   	/* will hold data for all nal */
		bs_t  bs;
		x264_nal_t  *nal;
	} out;

	uint8_t *nal_buffer;
	int    nal_buffer_size;
	int    reconfig;
	x264_t  *reconfig_h;

	/**** thread synchronization starts here ****/
	/* frame number/poc */
	int    i_frame;				// 编码帧号，用于计算POC（picture of count 标识视频帧的解码顺序）
	int    i_frame_num;
	int    i_thread_frames;			// Number of different frames being encoded by threads;
							// 1 when sliced-threads is on.
	int         i_nal_type;			// Nal单元的类型，这些类型值定义在x264.h中，需要理解的类型有：
							// 不分区（一帧作为一个片）非IDR图像的片；片分区A、片分区B、片分区C、
							// IDR图像中的片、序列参数集、图像参数集
	int         i_nal_ref_idc;			// Nal单元的优先级别,取值范围[0,1,2,3]，值越大表示优先级越高，此Nal单元就越重要
	int64_t   i_disp_fields;			// Number of displayed fields (both coded and implied via pic_struct)
	int         i_disp_fields_last_frame;
	int64_t   i_prev_duration; 		// Duration of previous frame
	int64_t   i_coded_fields; 		// Number of coded fields (both coded and implied via pic_struct)
	int64_t   i_cpb_delay;			// Equal to number of fields preceding this field since last buffering_period SEI
	int64_t   i_coded_fields_lookahead;	// Use separate counters for lookahead
	int64_t   i_cpb_delay_lookahead;
	int64_t   i_cpb_delay_pir_offset;
	int64_t   i_cpb_delay_pir_offset_next;
	int64_t   i_last_idr_pts;
	int         b_queued_intra_refresh;
	int         i_idr_pic_id;

	/* quantization matrix for decoding, [cqm][qp%6][coef] */
	int       (*dequant4_mf[4])[16];   			/* [4][6][16] */
	int       (*dequant8_mf[4])[64];   			/* [4][6][64] */
	/* quantization matrix for trellis, [cqm][qp][coef] */
	int      (*unquant4_mf[4])[16];   			/* [4][QP_MAX_SPEC][16] */
	int      (*unquant8_mf[4])[64];   			/* [4][QP_MAX_SPEC][64] */
	/* quantization matrix for deadzone */
	udctcoef        (*quant4_mf[4])[16];     		/* [4][QP_MAX_SPEC][16] */
	udctcoef        (*quant8_mf[4])[64];     		/* [4][QP_MAX_SPEC][64] */
	udctcoef        (*quant4_bias[4])[16];   		/* [4][QP_MAX_SPEC][16] */
	udctcoef        (*quant8_bias[4])[64];   		/* [4][QP_MAX_SPEC][64] */
	udctcoef        (*quant4_bias0[4])[16];  		/* [4][QP_MAX_SPEC][16] */
	udctcoef        (*quant8_bias0[4])[64];  		/* [4][QP_MAX_SPEC][64] */
	udctcoef        (*nr_offset_emergency)[4][64];

	/* mv/ref cost arrays. */
	uint16_t *cost_mv[QP_MAX+1];
	uint16_t *cost_mv_fpel[QP_MAX+1][4];

	/* includes both the nonlinear luma->chroma mapping and chroma_qp_offset */
	const uint8_t   *chroma_qp_table;

	/* Slice header，条带头结构体定义在common/common.h中 */
	x264_slice_header_t sh;

	/* SPS / PPS，我们只是用一个sps和一个pps */
	x264_sps_t   sps[1];
	x264_pps_t   pps[1];

	/* Slice header backup, for SEI_DEC_REF_PIC_MARKING */
	int   b_sh_backup;
	x264_slice_header_t sh_backup;

	/* cabac context */
	x264_cabac_t   cabac;

	/* 指示和控制帧编码过程的结构 */
	struct
	{
		/* Frames to be encoded (whose types have been decided) */
		/* 这个结构体涉及到X264编码过程中的帧管理，理解这个结构体中的变量在编码标准的理论意义是非常重要的 */
		x264_frame_t **current;	// 已确定帧类型，待编码帧，每一个GOP在编码前，每一帧的类型在编码前已经确定。
							// 当进行编码时，从这里取出一帧数据
		/* Unused frames: 0 = fenc, 1 = fdec */
		x264_frame_t **unused[2];	// 这个数组用于回收那些在编码中分配的frame空间，
							// 当有新的需要时，直接拿过来用，不用重新分配新的空间，提高效率
		/* Unused blank frames (for duplicates) */
		x264_frame_t **blank_unused;

		/* frames used for reference + sentinels */
		x264_frame_t *reference[X264_REF_MAX+2];	// 参考帧队列，注意参考帧都是重建帧

		int  i_last_keyframe; 		// Frame number of the last keyframe，上一个关键字的帧序号
		int  i_last_idr;            		// Frame number of the last IDR (not RP)，上一个IDR帧的帧序号，
							// 配合前面的i_frame，可以用来计算POC
		int  i_poc_last_open_gop; 	// Poc of the I frame of the last open-gop. The value is only assigned during the
							// period between that I frame and the next P or I frame, else -1
		int  i_input;    			// Number of input frames already accepted，已经接收的输入帧数
							// frames结构体中i_input指示当前输入的帧的（播放顺序）序号。
		int  i_max_dpb;  		// Number of frames allocated in the decoded picture buffer
		int  i_max_ref0;			// 最大前向参考帧数量
		int  i_max_ref1;			// 最大后向参考帧数量
		int  i_delay;   			// Number of frames buffered for B reordering；缓存的帧数，用于B帧重排序
		int  i_bframe_delay;
		int64_t i_bframe_delay_time;
		int64_t i_first_pts;
		int64_t i_prev_reordered_pts[2];
		int64_t i_largest_pts;
		int64_t i_second_largest_pts;
		int  b_have_lowres;  		// Whether 1/2 resolution luma planes are being used，亮度是否使用半像素精度
		int  b_have_sub8x8_esa;
	} frames;	// 指示和控制帧编码过程的结构

	/* current frame being encoded */
	x264_frame_t    *fenc;			// 指向当前编码帧

	/* frame being reconstructed */
	x264_frame_t    *fdec;			// 指向当前重建帧，重建帧的帧号要比当前编码帧的帧号小1

	/* references lists */
	int    i_ref[2];				// i_ref[0]和i_ref[1]分别表示前向和后向参考帧的数量
	int    b_ref_reorder[2];
	x264_frame_t    *fref[2][X264_REF_MAX+3];	// fref[0]和fref[1]分别表示存放前向和后向参考帧的数组（参考帧均是重建帧）
	x264_frame_t    *fref_nearest[2];

	/* hrd */
	int  initial_cpb_removal_delay;
	int  initial_cpb_removal_delay_offset;
	int64_t  i_reordered_pts_delay;

	/* Current MB DCT coeffs */
	struct
	{
        	ALIGNED_N( dctcoef luma16x16_dc[3][16] );
        	ALIGNED_16( dctcoef chroma_dc[2][8] );
        	// FIXME share memory?
        	ALIGNED_N( dctcoef luma8x8[12][64] );
        	ALIGNED_N( dctcoef luma4x4[16*3][16] );
	} dct;

	/* MB table and cache for current frame/mb */
	struct
	{
		int     i_mb_width;
		int     i_mb_height;
		int     i_mb_count;                 /* number of mbs in a frame */

		/* Chroma subsampling */
		int     chroma_h_shift;
		int     chroma_v_shift;

		/* Strides */
		int     i_mb_stride;
		int     i_b8_stride;
		int     i_b4_stride;
		int     left_b8[2];
		int     left_b4[2];

		/* Current index */
		int     i_mb_x;
		int     i_mb_y;
		int     i_mb_xy;
		int     i_b8_xy;
		int     i_b4_xy;

		/* Search parameters */
		int     i_me_method;
		int     i_subpel_refine;
		int     b_chroma_me;
		int     b_trellis;
		int     b_noise_reduction;
		int     b_dct_decimate;
		int     i_psy_rd;			 /* Psy RD strength--fixed point value*/
		int     i_psy_trellis; 		/* Psy trellis strength--fixed point value*/
		int     b_interlaced;
		int     b_adaptive_mbaff; 	/* MBAFF+subme 0 requires non-adaptive MBAFF i.e. all field mbs */

		/* Allowed qpel MV range to stay within the picture + emulated edge pixels */
		int     mv_min[2];
		int     mv_max[2];
		int     mv_miny_row[3]; 	/* 0 == top progressive, 1 == bot progressive, 2 == interlaced */
		int     mv_maxy_row[3];

		/* Subpel MV range for motion search. same mv_min/max but includes levels' i_mv_range. */
		int     mv_min_spel[2];
		int     mv_max_spel[2];
		int     mv_miny_spel_row[3];
		int     mv_maxy_spel_row[3];

		/* Fullpel MV range for motion search */
		ALIGNED_8( int16_t mv_limit_fpel[2][2] ); 	/* min_x, min_y, max_x, max_y */
		int     mv_miny_fpel_row[3];
		int     mv_maxy_fpel_row[3];

		/* neighboring MBs */
		unsigned int i_neighbour;
		unsigned int i_neighbour8[4];		/* neighbours of each 8x8 or 4x4 block that are available */
		unsigned int i_neighbour4[16];		/* at the time the block is coded */
		unsigned int i_neighbour_intra; 		/* for constrained intra pred */
		unsigned int i_neighbour_frame;		/* ignoring slice boundaries */
		int     i_mb_type_top;
		int     i_mb_type_left[2];
		int     i_mb_type_topleft;
		int     i_mb_type_topright;
		int     i_mb_prev_xy;
		int     i_mb_left_xy[2];
		int     i_mb_top_xy;
		int     i_mb_topleft_xy;
		int     i_mb_topright_xy;
		int     i_mb_top_y;
		int     i_mb_topleft_y;
		int     i_mb_topright_y;
		const x264_left_table_t *left_index_table;
		int     i_mb_top_mbpair_xy;
		int     topleft_partition;
		int     b_allow_skip;
		int     field_decoding_flag;

		/**** thread synchronization ends here ****/
		/* subsequent variables are either thread-local or constant,
		 * and won't be copied from one thread to another */

		/* mb table */
		uint8_t *base; 		/* base pointer for all malloced data in this mb */
		int8_t  *type;  		/* mb type */
		uint8_t *partition;  	/* mb partition */
		int8_t  *qp;			/* mb qp */
		int16_t *cbp;					// mb cbp:0x0?:luma, 0x?0:chroma, 0x100:luma dc,
									// 0x0200 and 0x0400: chroma dc(all set for PCM)
		int8_t  (*intra4x4_pred_mode)[8]; 	/* intra4x4 pred mode. for non I4x4 set to I_PRED_4x4_DC(2) */
									/* actually has only 7 entries; set to 8 for write-combining optimizations */
		uint8_t (*non_zero_count)[16*3];		/* nzc. for I_PCM set to 16 */
		int8_t  *chroma_pred_mode; 		/* chroma_pred_mode. cabac only. for non intra I_PRED_CHROMA_DC(0) */
		int16_t (*mv[2])[2];				/* mb mv. set to 0 for intra mb */
		uint8_t (*mvd[2])[8][2];			// absolute value of mb mv difference with predict, clipped to [0,33].
									// set to 0 if intra. cabac only
		int8_t   *ref[2]; 					/* mb ref. set to -1 if non used (intra or Lx only) */
		int16_t (*mvr[2][X264_REF_MAX*2])[2];/* 16x16 mv for each possible ref */
		int8_t  *skipbp; 				/* block pattern for SKIP or DIRECT (sub)mbs. B-frames + cabac only */
		int8_t  *mb_transform_size; 	/* transform_size_8x8_flag of each mb */
		uint16_t *slice_table; 			/* sh->first_mb of the slice that the indexed mb is part of
								 * NOTE: this will fail on resolutions above 2^16 MBs... */
		uint8_t *field;

		/* buffer for weighted versions of the reference frames */
		pixel *p_weight_buf[X264_REF_MAX];


	}
};
```
